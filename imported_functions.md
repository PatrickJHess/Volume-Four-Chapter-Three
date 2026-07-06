



# 🛠️ Imported Functions

:::{dropdown} Click to see `FEDInvest`
```python
def FEDInvest(price_date):
  """
    Fetches historical security prices from the FedInvest portal.


    Args:
        price_date (datetime.date): The date for which to retrieve prices.
            Note: Current day is typically available after 1:00 PM ET on business days.




    Returns:
        tuple: (pandas.DataFrame, str) if successful. The DataFrame contains
               security details (CUSIP, Price, Yield), and the string is the
               official "Prices For" date stamp from the site.
        tuple: (str, None) if the request fails or no data is found for the date
                (attempt to fetch current day before 1:00 PM ET).


    Example:
        >>> from datetime import date
        >>> df, stamp = FEDInvest(date(2025, 3, 17))
  """


  def make_date(date_value):
    # datetime64 are conerted
    if not isinstance(date_value,(datetime,date)):
      try:
        date_value=pd.Timestamp(date_value).date()
      except Exception as e: # Catch anything else unexpected
        print(f"wrong type for settlement or maturity {e}")


      date_value=pd.Timestamp(settlement).date()
    # convert timestamps and datetimes to date
    else:
      try:
        date_value=date_value.date()
      except:
        pass
    return date_value


  price_date=make_date(price_date)
  # make share date of prices and settlement date are settlement dates
  price_date=adjust_bond_pay_dates(price_date)[0]
  if price_date > date.today():
    return "price_date is in the future", None, None


  settlement_date=price_date+relativedelta(days=1)
  settlement_date=adjust_bond_pay_dates(settlement_date)


  # URL address of Treasury Direct Select A Date
  url = "https://treasurydirect.gov/GA-FI/FedInvest/selectSecurityPriceDate"


  # Standard headers to look like a real browser
  headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36\
     (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Content-Type": "application/x-www-form-urlencoded"
  }
  #  variable names and type identified from inspecting url
  month=str(price_date.month)
  day=str(price_date.day)
  year=str(price_date.year)


  # payload passed in request post
  payload={'priceDate.month':month,
           'priceDate.day':day,
           'priceDate.year':year,
           "submit": "Show Prices"}


  # fires off form and returns prices for date
  try:
        response = requests.post(url, data=payload, headers=headers, timeout=10)
        response.raise_for_status()
  except requests.exceptions.RequestException as e:
        return f"Connection Error: {e}", None


  # reads the html
  # Pandas recommends to wrap the response in StingIO to make file like
  tables=pd.read_html(StringIO(response.text),match='CUSIP')


  # from inspection there is a single table
  df=tables[0]


  df['MATURITY DATE']=pd.to_datetime(df['MATURITY DATE'])


  # drop rows equal to or less than settlement date
  df_filtered=df[df['MATURITY DATE']>pd.to_datetime(settlement_date[0])]


  return df_filtered, price_date,settlement_date[0]

```
:::

:::{dropdown} Click to see `clean_FEDInvest`
```python
def clean_FEDInvest(df):


    import pandas as pd
    # Filters for Standard Securities
    keep_rows=df['SECURITY TYPE'].str.contains('bill|note|bond',case=False)
    security_df=df[keep_rows].copy()
 
    # Removes Clutter
    drop_columns=['CUSIP','CALL DATE']
    security_df.drop(columns=drop_columns,inplace=True)


    # Creates a Time-Series Index
    security_df.set_index('MATURITY DATE',inplace=True)
    security_df.index=pd.to_datetime(security_df.index)
    security_df.sort_index(inplace=True)


    # Standardizes Financial Terms
    change_column_names={'RATE':'Coupon',
                         'BUY':'Price Ask',
                         'SELL':'Price Bid'}
    security_df.rename(columns=change_column_names,inplace=True)


    # Formats Numeric Data
    numeric_cols = ['Coupon', 'Price Ask', 'Price Bid', 'YIELD']
    for col in numeric_cols:
        if col in security_df.columns:
            security_df[col] = security_df[col].astype(str).str.replace('%', '', regex=False).astype(float)


    return security_df

```
:::


:::{dropdown} Click to see `create_payoff_df`

```py
def create_payoff_df(df, settlement,OLS=False):
    adjusted_maturities = adjust_bond_pay_dates(list(df.index))
    all_maturities = set(adjusted_maturities)


    df_payoff_columns = sorted(all_maturities)
    df_payoff_index=[i for i in range(len(df.index))]


    df_payoff = pd.DataFrame(
        np.zeros((len(df), len(df_payoff_columns))),
        columns=df_payoff_columns,
        index=df_payoff_index
    )
    total_rows = len(df)
    # Define a clean, pleasing HTML template for our status box
    def status_box(current, total):
        return f"""
        <div style="font-family: Arial, sans-serif; padding: 10px 15px; background-color: #f8f9fa; 
                    border-left: 4px solid #007bff; border-radius: 4px; width: fit-content; color: #333;">
            <b style="color: #007bff;">⚙️ Processing Bonds:</b> {current} of {total} added to DataFrame
        </div>
        """
    
    # initial display showing 0 bonds added
    progress_ui = display(HTML(status_box(0, total_rows)), display_id=True)


    for index,(maturity, coupon) in enumerate(zip(df.index, df['Coupon'])):


        # bond_pay_data returns payment dates and amounts
        row_pay_data = bond_pay_data(maturity, coupon, settlement=settlement)


        # Find any dates that aren't already columns
        new_dates = set(row_pay_data[0].flatten()) - all_maturities


        if new_dates:
          if OLS:
            df_clean = df_payoff.loc[(df_payoff != 0).any(axis=1),
                                         (df_payoff != 0).any(axis=0)]
            print("\u2705 DataFrame Complete (Exited Early)!")
            return df_clean
          else:
            # "\u2705 FIX: Add new dates to our master set and reindex
            all_maturities.update(new_dates)
            df_payoff = df_payoff.reindex(columns=sorted(all_maturities), fill_value=0.0)
 
        #    fill up the columns
        df_payoff.loc[index, row_pay_data[0]] = row_pay_data[1]
         # update the progress bar
        progress_ui.update(HTML(status_box(index + 1, total_rows)))
    # re-sort the columns so dates are chronological
    df_payoff = df_payoff.reindex(sorted(df_payoff.columns), axis=1)
    progress_ui.update(HTML("""
        <div style="font-family: Arial, sans-serif; padding: 10px 15px; background-color: #e6f4ea; 
                    border-left: 4px solid #34a853; border-radius: 4px; width: fit-content; color: #137333;">
            <b>"\u2705 DataFrame Complete!</b> All bonds added successfully.
            </div>
            """))
    return df_payoff

```
:::

:::{dropdown} Click to see `ns_spot_rates`

```py
def ns_spot_rates(interim_estimates,mat_years,sofr_rate=None):
  """
    Calculates spot rates using the Nelson-Siegel yield curve model.
    
    Handles both the unrestricted 4-parameter model and a restricted 
    3-parameter model where the short end is pinned to a proxy rate (like SOFR).


    Args:
        interim_estimates (array-like): Current parameter estimates. 
            Expects 3 values (Beta 0, Beta 2, Tau) if sofr_rate is provided.
            Expects 4 values (Beta 0, Beta 1, Beta 2, Tau) if unrestricted.
        mat_years (array-like): Time to maturity for each cash flow, in years.
        sofr_rate (float, optional): The short-term rate to pin the curve to. 
            Defaults to None (triggers unrestricted 4-parameter model).


    Returns:
        tuple:
            - np.ndarray: The calculated spot rates for the given maturities.
            - np.ndarray: The adjusted time-to-maturity array (zeros replaced with 1e-8).
  """
  # t saves typing
  t=mat_years


  # Avoid division by zero for t=0
  t = np.where(t == 0, 1e-8, t)
   
  if sofr_rate is not None:  # SOFR ties download the short rate


    # current values of estimates
    b_0,b_2,tau=interim_estimates
 
    # Restricted Nelson-Siegel model Restricted
    spot_rates = (
            b_0 
            + (sofr_rate - b_0) * (1 - np.exp(-t/tau)) / (t/tau) 
            + b_2 * ((1 - np.exp(-t/tau)) / (t/tau) - np.exp(-t/tau)))
  else:    # SOFR ignored


    # Unrestricted Nelson-Siegel model Unrestricted


    # current values of estimates
    b_0,b_1,b_2,tau=interim_estimates


    spot_rates = (
            b_0 
            + b_1 * (1 - np.exp(-t/tau)) / (t/tau) 
            + b_2 * ((1 - np.exp(-t/tau)) / (t/tau) - np.exp(-t/tau))
        )
  # pass these rates to the objective function for step one
  return spot_rates,t

```
:::

:::{dropdown} Click to see `calc_ytm`

```py
def calc_ytm(guesses, cash_flows, mat_years, prices):
    """
    Calculates the Yield to Maturity (YTM) using the Newton-Raphson method.
    """
    # use np.isscalar to check if the user passed a single number instead of an array
    if np.isscalar(guesses):
        # If a single guess is provided, broadcast it to match the length of the prices array
        guesses = [guesses] * len(prices)

    def ytm_objective(y, cash_flows, times, market_price):
        """Objective function: difference between modeled PV and actual price."""
        # Continuous compounding: PV = CF * e^(-y * t)
        pv = sum([cf * np.exp(-y * t) for cf, t in zip(cash_flows, times)])
        return pv - market_price

    def ytm_derivative(y, cash_flows, times, market_price):
        """Derivative of the objective function with respect to the yield (y)."""
        # Power rule applied to e^(-y * t) brings down the (-t)
        return sum([-t * cf * np.exp(-y * t) for cf, t in zip(cash_flows, times)])
    
    # Run the Newton-Raphson optimization
    ytm = optimize.newton(
        func=ytm_objective,
        x0=guesses,
        fprime=ytm_derivative,
        args=(cash_flows, mat_years, prices)
    )
    
    return ytm
```
:::


