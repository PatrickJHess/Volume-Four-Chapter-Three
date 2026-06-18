# Financial Python
## Volume: Pricing And Interest Rate Risk
## Chapter Three: Yield To Maturity And The Term Structure Of Interest Rates

### **Reality, Arbitrage, and Valuation**

Let’s remind ourselves what actually trades in the fixed-income market. Investors cannot purchase a "par yield," nor can they directly buy the "term structure." These are theoretical constructs used to map the market. In reality, investors buy individual bonds—discrete, messy packages of future cash flows. Because you are buying a pre-packaged bundle of cash flows, the market relies on a fundamental rule to determine a bond's fair price: **The Law of One Price**.

The Law of One Price dictates that two assets producing the exact same future cash flows must sell for the exact same current price. If a bond's market price drifted away from the true value of its underlying cash flows, arbitrageurs would buy the "cheap" cash flows and sell the "expensive" ones, locking in a risk-free profit until market forces snapped the bond's price back into alignment.

Therefore, arbitrage enforces our familiar valuation formula. A bond's price *must* exactly equal the sum of its individual cash flows, with each payment discounted by the specific term structure spot rate ($\text{PV}(t)$) for that exact point in time:


$$\text{Value Of Bond} =\sum_t^{T}\text{Pay Off}_t \times e^{-r(t) \times t}$$


Arbitrage establishes bond price, the yield to maturity (YTM) converts the prices to a single, hypothetical spot rate that can be used to solve the valuation formula:

$$r(t) = \text{YTM}  \quad t \in \{1, 2, 3, \dots, T\}$$

$$\text{Value Of Bond} =\sum_t^{T}\text{Pay Off}_t\times  e^{\textstyle -\mathrm{YTM}\times t}$$

### The "Non-Linear Combination" (The Coupon Effect)

Because of the exponent in the valuation formula ($e^{-\mathrm{YTM} \times t}$), cash flows occurring further into the future are mathematically penalized—discounted much more heavily than near-term cash flows. YTM acts as a complex, time-weighted average of the underlying spot rates-two bonds with the exact same maturity will often have different Yields to Maturity:

  * A High-Coupon Bond: Pays out a significant amount of cash early in its life. Its YTM is heavily influenced by the short-term spot rates on the term structure.

   * A Zero-Coupon Bond: Pays nothing until the very end. Its YTM is the spot rate for its specific maturity date.

If a 10-year bond has a higher YTM than another 10-year bond, it does not necessarily mean it is a "better deal"—different coupon rates simply imply different weights for the respective underlying spot rates resulting in different YTMs
