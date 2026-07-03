# 📈 ➡️ 🎯 The Term Structure Determines Yields To Maturity

## 💡🎯 What Is The Yield To Maturity And Why It Matters

The Yield to Maturity ($\text{ytm}$) of a bond is defined as the single discount rate ($\text{ytm}$) that equates the present value of the bond's expected future cash flows (coupons and principal) to its current market value.

## 🧮📝 Formula and Calculation

The general formula for a bond's value is:

$$\text{Bond Value} = \sum_{t=1}^{T}\text{Cash Flow}_{t}\times e^{-r_t\times t}$$

The discount rate at $t$ is $r_t$. Substituting the yield to maturity ($\text{ytm}$) for each discount rate:

$$\text{Bond Value} = \sum_{t=1}^{T}\text{Cash Flow}_{t}\times e^{-\text{ytm}\times t}$$

For a zero-coupon bond (a bond making a single payment at maturity), $\text{ytm}$ can be calculated directly:

$$P(t) = e^{-\text{ytm}\times t}\times \text{Cash Flow}_t$$

## ⚠️🚧 Limitations of Yield To Maturity

The yield to maturity ($\text{ytm}$) is often, but incorrectly, assumed to be the actual internal rate of return (IRR) an investor will earn by holding a bond until maturity. This assumption holds true only for default-free bonds that involve a single payment at maturity.

For bonds with multiple cash flows (i.e., coupon payments), the $\text{ytm}$ is merely an approximation because the reinvestment rates for these payments are uncertain and are unlikely to equal the yield to maturity. This uncertainty parallels the limitations investors encounter when using IRR in capital budgeting. Furthermore, the promised payments for bonds with default risk add an additional level of uncertainty.

Despite these limitations, it is important to recognize that bond pricing inherently incorporates the prevailing term structure of interest rates and reflects forward rates that are indicative of future reinvestment returns.

## 🔄⚙️ Calculating The Yield To Maturity

As noted above, the $\text{ytm}$ of a bond making a single payment may be directly calculated. There is no simple calculation for bonds that make multiple payments. A trial and error process is required. A practical way to implement the technique is a comparison of the errors of two initial guesses and then adjusting subsequent guesses based on the average rate of change between the errors and the guesses.

For functions that are differentiable, a more mathematically efficient version of this trial-and-error technique exists: the Newton-Raphson method, that utilizes a first-order Taylor series approximation of the function. 

$$\text{f}(x+\Delta x) \approx \text{f}(x) + \frac{d\text{f}(x)}{dx}\times\Delta x$$

The first derivative of the function is $\frac{d\text{f}(x)}{dx}$ at $x$, and $\Delta x$ is the change in $x$.

As $\Delta x$ goes to zero, the error of the approximation goes to zero.

The Newton-Raphson method improves each guess by scaling the error by the first derivative of the function at the previous guess:

$$\text{next guess} = \text{previous guess} + \frac{\text{f}(x)-\text{f}(\text{previous guess})}{\frac{df(\text{previous guess})}{dx}}$$

<br><br>

The first derivative of the function is:
$$\large\ -\sum_{t=1}^{T} t\times\text{Cash Flow}_t\times e^{-\text{rate}\times t}$$
<br><br>

## 📊🏦 A Three Bond Example

The yield to maturity on all three bonds is assumed to be 5% per annum. Coupon payments are semi-annual. The initial guess for each bond is 4%. 

The prices of the bonds at the actual spot rate of 5% and the initial guess of 4% are calculated and shown in the following table.

| Bond | Price At 5% | Price At 4% |
| :--- | :--- | :--- |
| six-month zero coupon | $97.53 | $98.02 |
| one-year zero coupon | $95.12 | $96.08 |
| one-year coupon rate 5 | $99.94 | $100.93 |

The pricing errors are scaled by the derivatives evaluated at the initial guess of 4%. The next guess is augmented by the scaled pricing errors.

| Bond | Pricing Error | Derivative | Delta Guess | Updated Guess |
| :--- | :--- | :--- | :--- | :--- |
| six-month zero coupon | -$0.49 | -49.00 | 1.0% | 5.0% |
| one-year zero coupon | -$0.96 | -96.88 | 1.0% | 5.0% |
| one-year coupon rate 5 | -$0.99 | -99.71 | 1.0% | 5.0% |

For all three bonds a solution is reached within one iteration.

Although this example includes only a few payment dates with easy solutions, more complex bonds require more calculations, and for that, there is a SciPy function that solves for the yield to maturity. We'll put it to work in the notebook *Yield To Maturity And The Term Structure Of Interest Rates*.

:::{tip} ✨ AI Study Assistant
Select a prompt below to open Google AI and explore more concepts about the term sturcuture and yields to maturity *(Sign in to save your chat history!)*
---

✨ [**Because yields to maturity are derived from the term structure, two bonds with the same maturity will the same yield to maturity. ↗**](https://www.google.com/search?udm=50&q=Because+yields+to+maturity+are+derived+from+the+term+structure,+Treasury+bond+with+the+same+maturity+have+the+same+yield+to+maturity.)

---

✨ [**A colleague told me that the yield to maturity of a bond never equals the internal rate of return from owning until it matures↗**](https://www.google.com/search?udm=50&q=A+colleague+told+me+that+the+yield+to+maturity+of+a+bond+neve+equals+the+internal+rate+of+return+from+owning+until+it+matures)---

---

✨ [**The yield to maturity on long-term bonds mostly depend upon their principal payment. ↗**](https://www.google.com/search?udm=50&q=The+yields+to+maturity+on+long-term+bonds+mostly+depend+upon+their+principal+payments.)


---

✨ [**Is the yield to maturity curve the same as the par yield curve? ↗**](https://www.google.com/search?udm=50&q=Is+the+yield+to+maturity+curve+the+same+as+the+par+yield+curve?)

:::