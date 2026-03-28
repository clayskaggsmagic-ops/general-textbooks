# Chapter 3: Deal Mechanics

## SAFEs, Convertible Notes, Priced Rounds, and How to Read a Term Sheet

---

### Why This Chapter Will Save You Money

The instruments described in this chapter are the literal documents that determine how much of your
company you're giving away, to whom, and under what conditions. Every clause has a history, a purpose,
and — in many cases — a trap. Founders who don't understand these documents rely entirely on their
lawyers. That's fine to a point, but your lawyer works for you, and an uninformed client is a poorly
served client.

By the end of this chapter, you'll be able to pick up a SAFE, a convertible note, or a term sheet
and understand every material provision. You won't need to become a securities lawyer, but you'll
know enough to have an informed conversation with yours.

---

## 3.1 The Two Worlds: Convertible Instruments vs. Priced Rounds

Before we dig into specifics, understand that early-stage fundraising uses two fundamentally different
approaches:

**1. Convertible instruments (SAFEs and convertible notes):** The investor gives you money now. In
exchange, they receive the right to convert that money into equity later, typically at your next
priced round. No shares are issued today; no valuation is formally agreed upon today. The conversion
happens later, on terms partially set now and partially set later.

**2. Priced rounds:** The investor gives you money now. In exchange, you issue them shares (usually
preferred stock) at a specific price per share, based on a valuation negotiated now. Shares are
issued today. Everything is agreed upon today.

**When each is used:**

| Factor                | Convertible (SAFE/Note) | Priced Round        |
|-----------------------|------------------------|---------------------|
| Stage                 | Pre-seed, seed          | Seed (sometimes), Series A+ |
| Amount raised         | Under $2M              | Over $2M (typically) |
| Legal cost            | $0–$2,000              | $10,000–$50,000     |
| Complexity            | Low                    | High                |
| Time to close         | Days to weeks          | Weeks to months     |
| Board impact          | Usually none           | Investor board seat |
| Investor protections  | Minimal                | Extensive           |

At the pre-seed stage, you'll almost certainly use SAFEs or convertible notes. At seed, it's a coin
flip — some rounds are SAFE-based, others are priced. At Series A and beyond, it's almost always a
priced round.

---

## 3.2 SAFEs: The Simple Agreement for Future Equity

### Origin Story

The SAFE was created by Y Combinator in 2013 to solve a problem: raising small amounts of early-stage
capital was too expensive and slow. Convertible notes (the instrument that SAFEs replaced in many
contexts) were based on debt — they had interest rates, maturity dates, and legal complexity derived
from loan agreements. Y Combinator's lawyers designed the SAFE as a simpler alternative: not debt,
not equity, but a contractual right to receive equity in the future.

Today, the SAFE is the dominant instrument for pre-seed and many seed-stage raises. The standard SAFE
documents are publicly available on Y Combinator's website, and most investors and lawyers are deeply
familiar with them.

### How a SAFE Works: The Basics

A SAFE is a contract between you (the company) and an investor. The investor gives you cash. You give
the investor a promise: when you next sell shares in a priced round (called a "qualified financing"),
the investor's cash will convert into shares at that time, using a price that's favorable to the
investor (because they invested earlier, when the risk was higher).

The SAFE doesn't have:
- A maturity date (it doesn't expire)
- An interest rate (it's not a loan)
- Share issuance at the time of investment (no shares are created yet)
- A voting right (SAFE holders are not shareholders)

The SAFE does have:
- A conversion mechanism (how the cash turns into shares)
- A valuation cap and/or discount (how the investor gets a better price than later investors)
- Provisions for what happens in special situations (company gets acquired, company dissolves, etc.)

### The Two Flavors: Pre-Money vs. Post-Money SAFEs

Y Combinator released an updated SAFE in 2018 called the **post-money SAFE.** This is a critical
distinction that many founders misunderstand.

**Post-Money SAFE (YC Standard, 2018+):**

The valuation cap on a post-money SAFE represents the **total value of the company after the SAFE
money is included.** This means the investor knows exactly what percentage of the company they're
buying.

Example:
- Investor invests $500K on a post-money SAFE with a $5M cap.
- The investor owns exactly $500K / $5M = 10% of the company (on a post-money basis), assuming the
  cap applies.

This is extremely clean and predictable. The investor knows their ownership percentage at the time of
investment. The catch: **each additional SAFE you sell dilutes the founders, not the previous SAFE
holders.** If you sell one SAFE at $500K on a $5M cap (10%) and then sell another at $500K on the
same $5M cap (another 10%), the founders are now diluted by 20%. The two SAFE holders are each at 10%.

**Pre-Money SAFE (Original YC SAFE, pre-2018):**

The valuation cap on a pre-money SAFE represents the company's value **before the SAFE money.** This
means the investor's exact percentage is determined later, when the SAFE converts, and depends on how
many other SAFEs were sold.

Example:
- Investor invests $500K on a pre-money SAFE with a $5M cap.
- Another investor invests $500K on a pre-money SAFE with the same $5M cap.
- At conversion: total SAFE investment is $1M. Pre-money cap is $5M. Post-money is $6M.
- Each investor owns $500K / $6M = 8.33%.

With the pre-money SAFE, all SAFE holders dilute each other. With the post-money SAFE, the founders
absorb all the dilution. Most lawyers and accelerators now use post-money SAFEs. As a founder, be
aware that post-money SAFEs can dilute you more than you expect if you sell multiple SAFEs — the
dilution stacks against you specifically.

### Valuation Cap

The **valuation cap** is the maximum effective valuation at which the SAFE converts. It's the
investor's upside protection.

How it works:
- You sell a SAFE with a $5M cap.
- Later, you do a Series A at a $20M pre-money valuation.
- The SAFE investor converts at the $5M cap, not the $20M valuation.
- This means they get 4x more shares per dollar than the Series A investors. This is fair — they
  invested earlier, took more risk, and deserve a better price.

If the Series A had been at a $3M pre-money valuation (lower than the cap), the SAFE would convert
at the Series A price. The cap is a ceiling, not a floor.

### Discount

The **discount** is an alternative (or additional) mechanism to give SAFE investors a better price.
Instead of a fixed cap, the investor gets a percentage discount on whatever price the next investors
pay.

Example:
- You sell a SAFE with a 20% discount (no cap).
- Later, you do a Series A at $10/share.
- The SAFE investor converts at $10 × (1 - 0.20) = $8/share.
- They get 25% more shares per dollar than the Series A investors.

### Cap vs. Discount: Which Is Better?

Most SAFEs have a **cap only** or a **cap and discount** (the investor gets whichever produces a
better price). SAFEs with a discount only (no cap) are less common.

As a founder:
- **Cap only** is the cleanest. You know exactly how much dilution you're taking.
- **Cap + discount** is more investor-friendly. The investor gets the better of the two conversions.
  If your valuation goes way up, the cap protects them. If it goes up modestly, the discount kicks
  in.
- **Discount only** is risky for you as a founder if your next-round valuation is high, because the
  investor gets a percentage off whatever price you close at.

### Most Favored Nation (MFN) Clause

An MFN clause says: if you sell a SAFE to someone else on better terms (lower cap, higher discount),
the existing SAFE investor can switch to those better terms.

This protects early investors from being disadvantaged if you sell a later SAFE at a lower cap. It's
standard and generally reasonable.

### What Happens When the SAFE Converts

SAFEs convert in three scenarios:

**1. Qualified Financing (the normal case).** You raise a priced round. The SAFE converts into shares
of the same class as the new round (usually Series A Preferred Stock), at a price determined by the
cap and/or discount.

**2. Liquidity Event (acquisition).** If the company is acquired before a priced round, the SAFE
holder can either:
- Convert into common stock using the cap, and participate in the acquisition payout; or
- Get their money back (receive the amount they invested).

They choose whichever is better. In a big acquisition, conversion is better. In a fire sale,
getting their money back is better.

**3. Dissolution.** If the company shuts down, SAFE holders get their money back — but only after
all debts and obligations are paid. In practice, if a startup shuts down, there's usually nothing
left, and SAFE holders lose their investment.

### How Many SAFEs Can You Sell?

There's no legal limit, but there's a practical limit. Every SAFE you sell creates future dilution.
If you sell $2M in SAFEs on a $10M cap, that's 20% dilution at conversion. If you then sell another
$1M on a $10M cap, that's another 10%. You're now looking at 30% dilution before your priced round
even happens.

Excessive SAFE dilution creates two problems:
1. **Less room for the Series A investor.** If 30% of your company is already spoken for by SAFE
   holders, the A investor is buying into a pre-diluted cap table. They may demand a lower valuation
   to compensate.
2. **Less equity for you and your team.** After SAFE conversion, the option pool, and the Series A
   investment, your remaining ownership can be surprisingly low.

**Rule of thumb:** Keep total SAFE dilution under 20–25% to maintain a healthy cap table for your
priced round.

---

## 3.3 Convertible Notes: The Older Sibling

### What a Convertible Note Is

A convertible note is a **loan** that converts into equity. Unlike a SAFE, it's structured as debt.
This means it has features that SAFEs don't:

- **Interest Rate.** Typically 2–8%. The interest accrues and is added to the principal at conversion,
  giving the investor slightly more shares. The interest is not paid in cash — it just adds to the
  conversion amount.

- **Maturity Date.** The date by which the note must either convert or be repaid. Typically 18–24
  months. If you haven't done a priced round by the maturity date, the note is technically due. In
  practice, investors rarely demand repayment — they usually extend the maturity date or negotiate a
  conversion. But the maturity date gives them leverage.

- **Valuation Cap and/or Discount.** Same mechanics as SAFEs.

### Why Convertible Notes Still Exist

If SAFEs are simpler and cheaper, why would anyone use a convertible note?

1. **Some investors prefer the debt structure.** The maturity date gives them a forcing function: if
   you haven't raised a priced round by the maturity date, the conversation about what happens next
   gives the note holder leverage.

2. **Tax and accounting treatment.** In some jurisdictions, debt instruments have different tax
   treatment than SAFEs. This matters more to the investor than to you, but some investors have fund
   structures that prefer notes.

3. **Seniority in liquidation.** Because a convertible note is debt, it sits above equity in the
   capital structure. If the company fails, note holders have a stronger claim on remaining assets
   than SAFE holders. (In practice, there are rarely any assets, so this is mostly theoretical.)

4. **Tradition.** In many markets outside the Bay Area — New York, Boston, international — convertible
   notes are more common than SAFEs simply because they've been used longer and local investors are
   more familiar with them.

### The Maturity Date Problem

The most consequential difference between a note and a SAFE is the maturity date. Here's a scenario
that goes wrong:

You issue a convertible note with a 24-month maturity. At month 18, you haven't raised a priced round
yet — growth was slower than expected, and you're still working toward product-market fit.

The note is about to mature. Technically, the note holder can demand repayment. You don't have the
cash to repay. What happens?

**Three possible outcomes:**
1. **The note holder extends the maturity.** Most common. They agree to push the maturity out another
   12 months, perhaps with modified terms. But they hold leverage — they might demand a lower cap, a
   higher interest rate, or other concessions.

2. **The note converts at maturity.** Some notes have a provision that converts the note into equity
   automatically at a predetermined valuation if the maturity date arrives without a qualified
   financing. This is called a **conversion on maturity** or **maturity conversion.** The valuation
   is usually the cap. This is actually a reasonable outcome for everyone.

3. **The note holder demands repayment.** Rare, but it happens — especially if the relationship has
   deteriorated or the investor has lost confidence. If you can't repay, the note holder could
   theoretically force the company into bankruptcy. This is the nuclear option and almost nobody
   exercises it, but the threat is real.

**The takeaway:** If you use convertible notes, negotiate the most favorable maturity conversion terms
you can up front, and never let a maturity date approach without a plan.

---

## 3.4 SAFEs vs. Convertible Notes: A Head-to-Head Comparison

| Feature             | SAFE                        | Convertible Note              |
|---------------------|-----------------------------|-------------------------------|
| Legal structure     | Contract (not debt)         | Debt instrument (a loan)      |
| Interest rate       | None                        | 2–8% (accrues, doesn't pay)  |
| Maturity date       | None                        | 18–24 months                  |
| Conversion trigger  | Priced round                | Priced round or maturity      |
| Valuation cap       | Yes                         | Yes                           |
| Discount            | Sometimes                   | Sometimes                     |
| MFN clause          | Sometimes                   | Sometimes                     |
| Legal cost          | Near zero (standard docs)   | $2,000–$5,000                 |
| Balance sheet       | Not debt                    | Listed as debt (matters for   |
|                     |                             | accounting and some investors) |
| Investor leverage   | Lower                       | Higher (maturity/repayment)   |
| Founder-friendliness| Higher                      | Lower                         |

**Bottom line:** For most pre-seed and early seed raises, use a post-money SAFE. It's simpler,
cheaper, and industry-standard. Use a convertible note if your investors specifically require it or
if there's a strategic reason for the debt structure.

---

## 3.5 Priced Rounds: When Things Get Real

### What a Priced Round Is

A priced round is the "real" fundraise. You issue **preferred stock** — a new class of shares with
special rights and protections — at a specific price per share. This creates a formal valuation, a
detailed legal agreement (called the **investment documents** or **definitive agreements**), and
usually a shift in corporate governance (the investor gets a board seat).

Priced rounds are governed by a set of standard documents that have been refined over decades. The
most widely used standards are the **NVCA (National Venture Capital Association) model documents**
and the **Series Seed documents** (for smaller priced rounds).

### The Key Documents in a Priced Round

A priced round involves five to seven core documents. Here's what each one does:

**1. Term Sheet.** The non-binding summary of the deal's key terms. This is what you negotiate first.
Once both sides agree on the term sheet, the lawyers draft the definitive documents to implement it.
The term sheet is not legally binding (except for confidentiality and exclusivity provisions), but
reneging on a term sheet is considered extremely bad form and can damage your reputation.

**2. Stock Purchase Agreement (SPA).** The contract governing the sale of shares. It includes
representations and warranties (statements by the company that certain things are true — e.g., the
company is validly incorporated, there are no lawsuits pending, the IP is properly assigned) and the
mechanics of the share purchase (how many shares, at what price, when payment is due).

**3. Certificate of Incorporation (Amended & Restated).** Your original certificate of incorporation
gets amended to create the new class of preferred stock and define its rights. This is a public
document filed with Delaware.

**4. Investors' Rights Agreement (IRA).** This agreement governs the ongoing rights of the investors.
It includes:
- **Registration rights:** The right to force the company to register their shares for public sale
  (relevant at IPO).
- **Information rights:** The right to receive regular financial reports from the company.
- **Pro-rata rights:** The right to participate in future funding rounds to maintain their ownership
  percentage.
- **Right of first refusal and co-sale agreement (ROFR/co-sale):** Restrictions on founders selling
  their shares without offering them to the investors first.

**5. Voting Agreement.** This agreement defines how the board of directors is composed and how certain
voting decisions are handled. It typically specifies:
- How many board seats the founders control
- How many the investors control
- How many are "independent" (filled by mutual agreement)
- Voting commitments by all shareholders regarding board composition

**6. Right of First Refusal and Co-Sale Agreement.** Sometimes a separate document, sometimes
incorporated into the IRA. It says that if a founder wants to sell shares, the company gets the first
right to buy them, then the investors get the right to buy them, and if neither exercises their right,
the investors can sell some of their shares alongside the founder ("tag along").

### Common vs. Preferred Stock

This is a critical distinction that determines who gets paid and in what order.

**Common stock** is what founders and employees hold. It's the basic ownership unit. Common
stockholders are the "residual claimants" — they get what's left after everyone else is paid.

**Preferred stock** is what investors get in a priced round. Preferred stockholders have special rights
that common stockholders don't:

- **Liquidation preference** (covered in detail below)
- **Anti-dilution protection** (covered below)
- **Protective provisions** (veto rights over certain actions)
- **Conversion rights** (the right to convert preferred to common, usually at a 1:1 ratio)
- **Dividend rights** (usually a stated dividend rate, but dividends are almost never actually paid
  at the startup stage)

The existence of preferred stock is why "valuation" can be misleading. If investors hold preferred
stock with a 1x liquidation preference and the company sells for an amount equal to the total invested
capital, the investors get all their money back and the common stockholders (founders, employees) get
nothing. The company might have had a "$50M valuation," but if it sells for $10M and investors put in
$10M, the preferred stockholders get $10M and common stockholders get $0.

---

## 3.6 The Term Sheet: A Line-by-Line Decoder

The term sheet is the most important document in a fundraise. Let's go through every material
provision.

### Valuation

- **Pre-money valuation:** The company's value before the new investment.
- **Post-money valuation:** Pre-money + new money raised.
- **The investor's ownership:** New money / Post-money = investor's percentage.

Already covered in Chapter 1, but it's on the term sheet. Note that the pre-money valuation is
"fully diluted," meaning it includes all outstanding shares PLUS the option pool. This matters
because of the option pool shuffle (Chapter 1, Section 1.8).

### Option Pool

The term sheet will specify the size of the employee option pool, usually as a percentage of the
post-money capitalization. Common range: 10–20%. As discussed, this pool is typically created from
the pre-money valuation, diluting the founders.

**Negotiation tip:** Push for a smaller option pool if you don't plan to hire aggressively before the
next round. A 10% pool is reasonable for an 18-month runway with modest hiring. A 20% pool is
excessive unless you plan to hire a lot of senior people. Any unused pool dilutes you for no reason.

### Liquidation Preference

This is the **single most important economic term** after valuation. It determines who gets paid
first — and how much — when the company has a "liquidity event" (sale, merger, or dissolution).

**How it works:**

Preferred stockholders have a liquidation preference that entitles them to receive a certain amount
back before common stockholders receive anything. The standard is **1x non-participating.**

**1x Non-Participating Preferred (the standard):**

The investor gets back either:
- (a) 1x their investment (the amount they put in), OR
- (b) their pro-rata share of the total proceeds (as if they'd converted to common stock)

They choose whichever is higher. They do NOT get both.

Example:
- Investors put in $5M for 25% of the company.
- **Scenario 1: Company sells for $8M.** The investors can choose: (a) $5M (their 1x), or (b) 25%
  of $8M = $2M. They choose (a): $5M. Founders and common holders split the remaining $3M.
- **Scenario 2: Company sells for $100M.** The investors can choose: (a) $5M (their 1x), or (b) 25%
  of $100M = $25M. They choose (b): $25M. Everyone else splits $75M pro-rata.

This is the fair, standard structure. The liquidation preference protects the investor's downside
(they get their money back in a modest exit) while allowing the founders to benefit from a big exit.

**Participating Preferred (the aggressive version — avoid if possible):**

With participating preferred, the investor gets BOTH:
- Their 1x (money back), AND
- Their pro-rata share of whatever remains

This is sometimes called **"double dipping."**

Example:
- Investors put in $5M for 25%.
- Company sells for $20M.
- Investors get $5M back first, leaving $15M.
- Investors also get 25% of $15M = $3.75M.
- Investors total: $8.75M. Founders total: $11.25M.
- Without participation, investors would have gotten 25% of $20M = $5M (or their $5M back). With
  participation, they get $8.75M.

Participating preferred significantly reduces the founders' payout in moderate exits. A company selling
for 3–5x the invested capital should be a good outcome for everyone, but participating preferred can
make it feel like a loss for the founders.

**Negotiation tip:** Insist on 1x non-participating. If an investor demands participating preferred,
negotiate a **participation cap** — a maximum return beyond which the participation feature switches
off (e.g., "participating up to 3x, then converts to common"). Some founders accept participating
preferred in exchange for a higher valuation, but this is usually a bad trade.

### Anti-Dilution Protection

Anti-dilution provisions protect investors if the company raises money at a lower valuation in the
future (a "down round"). If the next round is at a lower price per share, the anti-dilution provision
adjusts the investor's conversion ratio so they receive more shares, partially compensating them for
the valuation decline.

**Two types:**

**Full Ratchet:** The investor's conversion price is adjusted to match the lower price per share in
the down round, regardless of how many shares are sold at the lower price. This is extremely
aggressive and heavily dilutes the founders.

Example: Investor bought at $10/share. New round is at $5/share. Full ratchet adjusts the investor's
price to $5/share, doubling their shares. The founders absorb all the dilution.

**Weighted Average (Broad-Based):** The industry standard. The conversion price is adjusted based on
a formula that considers both the lower price and the number of shares issued at the lower price. The
adjustment is proportional to the size of the down round. A small down round barely affects the
conversion price; a large one has a bigger effect.

**Negotiation tip:** Broad-based weighted average is standard. Accept it. Reject full ratchet unless
you're in a position of extreme weakness. Full ratchet can destroy founder equity in a down round.

### Board Composition

The term sheet specifies the composition of the board of directors. Common structures:

**Early stage (seed/Series A):**
- 3-person board: 2 founders + 1 investor
- 5-person board: 2 founders + 1 investor + 2 independents

**Later stage:**
- 5-person board: 2 founders + 2 investors + 1 independent
- Variations abound

**Why it matters:** The board controls major decisions — hiring/firing the CEO, approving fundraises,
approving acquisitions, setting executive compensation. If investors control a majority of the board,
they can override the founders on any of these decisions. This is rare at the seed/Series A stage, but
becomes a real risk at later stages or in struggling companies.

**Negotiation tip:** Maintain founder control of the board as long as possible. At Series A, a 3-person
board (2 founders + 1 investor) is ideal. A 5-person board with 2 founders + 1 investor + 2
independents is also fine, as long as the independents are mutually agreed upon.

### Protective Provisions

These are **veto rights** that give investors the power to block certain corporate actions, even if
they don't control the board. Standard protective provisions require investor approval for:

- Issuing new shares or securities
- Taking on debt above a specified threshold
- Selling the company or substantially all of its assets
- Changing the certificate of incorporation
- Liquidating or dissolving the company
- Declaring or paying dividends
- Changing the number of board seats
- Creating a new class of stock with equal or greater rights

These are standard and reasonable. The investor put in millions of dollars; they should have a say in
existential decisions. But pay attention to the specifics:

- **Spending caps:** Some term sheets include a protective provision requiring investor approval for
  any expenditure above $X (say, $50K). This is micromanagement and should be resisted.
- **Hiring restrictions:** Some require approval for executive hires above a certain salary. This is
  also aggressive.
- **Scope of "sell the company":** Make sure this is defined clearly. You don't want a protective
  provision that requires investor approval to license your technology or enter a partnership.

### Pro-Rata Rights

Pro-rata rights give the investor the option to invest in future rounds to maintain their ownership
percentage. If an investor owns 10% and you raise a Series B, they can invest enough in the B round
to keep their 10%.

This is standard and generally harmless. In hot rounds, pro-rata rights from existing investors can
actually be helpful — it's a signal of confidence.

However, pro-rata rights can become a problem if you have too many investors all exercising their
rights. This reduces the space available for new investors in the next round.

### Information Rights

The right to receive financial information, typically:
- Annual financial statements
- Quarterly financial statements
- Monthly financial summaries (sometimes)
- Annual budget/operating plan

Standard for any investor putting in a meaningful amount. Not something to fight over.

### Exclusivity (No-Shop)

During the period between signing the term sheet and closing the round (typically 30–60 days), you
agree not to solicit or engage with other potential investors. This protects the lead investor from
doing due diligence while you're shopping for a better deal.

**Standard duration:** 30 days. Push back on anything longer than 45 days.

### Closing Conditions

The term sheet will list conditions that must be met before the deal closes:
- Satisfactory legal due diligence
- No material adverse changes in the business
- Board and stockholder approval
- Creation of the option pool
- Various deliverables (amended certificate of incorporation, stockholder agreements, etc.)

---

## 3.7 Due Diligence: What Investors Look At Before They Wire Money

### What Due Diligence Is

After a term sheet is signed, the investor and their lawyers conduct **due diligence** — a thorough
examination of your company's legal, financial, and operational health. This process takes 2–6 weeks
and often determines whether the deal actually closes.

Think of it like a home inspection after you've agreed on a price for a house. The inspector might
find problems that kill the deal or lead to a renegotiation.

### What They're Looking For

**Corporate structure.** Is the company properly incorporated? Are the bylaws in order? Have shares
been properly authorized and issued? Are the equity records accurate?

**Cap table.** Who owns what? Are all equity grants properly documented? Are there any outstanding
claims or disputes over equity?

**IP.** Are all IP assignment agreements in place? Did any founder create IP while employed elsewhere?
Are there any open-source license issues? Any patent claims?

**Contracts.** Customer contracts, vendor agreements, partnership agreements, employment agreements.
They're looking for unusual terms, liabilities, or obligations.

**Financial records.** Bank statements, revenue records, expense reports. Is the financial picture
consistent with what you represented during the pitch?

**Litigation.** Any pending or threatened lawsuits? Any regulatory investigations?

**Tax compliance.** Are all taxes paid? Are there any outstanding liabilities?

**Regulatory.** Is the company in compliance with applicable regulations? This is especially important
for companies in healthcare, fintech, education, and other regulated industries.

### The Data Room

You'll need to set up a **data room** — a secure online repository containing all of your company's
documents. Investors and their lawyers access this room to conduct due diligence.

Use a service like DocSend, Google Drive (with restricted sharing), or a specialized virtual data
room provider. Organize documents into clear folders:

- Corporate documents (incorporation, bylaws, board resolutions)
- Cap table and equity documents (stock purchase agreements, option grants, SAFEs)
- IP documents (assignment agreements, patent filings, license agreements)
- Financial documents (bank statements, P&L, revenue data)
- Contracts (customer agreements, vendor contracts, leases)
- Employment (offer letters, employee agreements, contractor agreements)
- Tax (tax returns, EIN documents, franchise tax receipts)
- Insurance (D&O policy, general liability)

Setting up the data room before you start fundraising dramatically speeds up the closing process. Many
founders are caught off guard by how long it takes to gather all the documents investors request.

---

## 3.8 Pro Tips: Negotiation Tactics for Term Sheets

### Understand Your Leverage

Your leverage in a term sheet negotiation comes from one and only one thing: **alternatives.** If you
have multiple term sheets, you have leverage. If you have one term sheet and you're running out of
runway, you have almost none.

This is why generating multiple term sheets simultaneously is so important. Even if you ultimately
prefer one investor, having a competing offer strengthens your negotiation position on every term.

### Pick Your Battles

You cannot win on every term. Experienced founders prioritize:

1. **Valuation / dilution:** Fight for this, but not at the expense of everything else.
2. **Liquidation preference:** Insist on 1x non-participating. This is the hill to die on.
3. **Board composition:** Maintain founder majority as long as possible.
4. **Option pool size:** Negotiate this down if you don't need a large pool immediately.
5. **Anti-dilution:** Broad-based weighted average is standard. Full ratchet is unacceptable.

Don't spend political capital negotiating information rights or registration rights. These are largely
standard and not worth the fight.

### Use Your Lawyer, But Don't Be Your Lawyer

Let your lawyer handle the legal language and the document drafting. But make the business decisions
yourself. Your lawyer should explain the implications of each term; you should decide which terms to
push on based on your priorities and leverage.

A common mistake: the founder's lawyer aggressively negotiates every clause to "protect the founder,"
alienating the investor in the process. By the time the documents are signed, the relationship is
adversarial rather than collaborative. Remember: you'll be working with this investor for years. Start
the relationship well.

### The "Standard" Argument

Both sides will frequently say "this is standard" to justify their preferred terms. Sometimes it's
true; sometimes it's a negotiating tactic. Having an experienced startup lawyer is essential for
distinguishing between genuinely standard terms and terms that only look standard.

Reference points: The NVCA model documents are publicly available and represent genuine market
standard. If a term deviates significantly from the NVCA model, ask why.

---

## 3.9 Side Letters and Secret Deals

### What Side Letters Are

A **side letter** is a separate agreement between you and a specific investor that grants them rights
or terms not available to other investors in the same round. Common side letter provisions:

- **Advisor equity:** The investor also receives a small advisor grant.
- **Board observer seat:** The investor gets to attend board meetings but not vote.
- **Additional information rights:** More detailed or frequent financial reports.
- **Pro-rata rights:** For smaller investors who wouldn't otherwise qualify.
- **Special follow-on rights:** The right to invest a specified amount in the next round.

### Why Side Letters Are Dangerous

Side letters can create a web of special deals that become impossible to manage:

- If Investor A has a side letter with a pro-rata right and Investor B has one with a super-pro-rata
  right, you're overcommitting allocation in future rounds.

- MFN clauses in other investors' agreements might entitle them to the same terms granted in any
  side letter, cascading the special deal to everyone.

- During due diligence for the next round, new investors will see all the side letters and may be
  concerned about the complexity.

**Best practice:** Minimize side letters. If you must grant one, understand the cascade effects (will
MFN clauses apply?), and disclose all side letters to your lawyer.

---

## 3.10 Closing the Round: The Wire and the Paperwork

### What Closing Looks Like

Closing is the moment when the legal documents are signed and the money is wired to your bank account.
In a priced round, closing can feel anticlimactic after weeks of negotiation — it's a series of
DocuSign signatures and a bank transfer.

**Steps to closing:**

1. **Term sheet signed.** Non-binding, but establishes the framework.
2. **Due diligence conducted.** Investor's lawyers review your documents.
3. **Definitive documents drafted.** Lawyers turn the term sheet into final legal documents (SPA, IRA,
   voting agreement, amended certificate of incorporation, etc.).
4. **Negotiations on documents.** Small details get hammered out. Representations and warranties are
   finalized.
5. **Board and stockholder approval.** Your board approves the new issuance. Existing stockholders
   consent to the amended certificate of incorporation.
6. **Signing.** All parties sign the definitive documents.
7. **Filing.** Amended certificate of incorporation filed with Delaware.
8. **Wire.** Money arrives in your account. (**Pro tip:** Open a second bank account at a different
   institution and move a portion of the funds there. Silicon Valley Bank's collapse in 2023 taught
   founders not to keep all their cash in one institution.)
9. **Post-closing.** Issue stock certificates (or book entries), update the cap table, file any
   necessary securities filings, and send out closing binders to all parties.

### Securities Law Basics

You might be wondering: "Isn't it illegal to sell securities without SEC registration?" Yes — but
startup fundraising relies on **exemptions** from SEC registration.

The most common exemption is **Regulation D, Rule 506(b),** which allows you to sell securities to
an unlimited number of **accredited investors** (individuals with a net worth over $1M excluding their
home, or annual income over $200K/$300K for couples) without registering with the SEC.

Requirements:
- No "general solicitation" — you can't publicly advertise that you're selling securities. Cold emails,
  social media posts, and public pitches at demo days are potential violations. (There are nuances —
  Rule 506(c) allows general solicitation with verification of accredited status.)
- You must file a **Form D** with the SEC within 15 days of your first sale. This is a simple
  disclosure form.
- Some states require additional filings (called "blue sky" filings).

Your lawyer handles all of this. But know that it exists — securities law violations can result in
investors having the right to rescind their investment (demand their money back), which is a nightmare
scenario.

---

## 3.11 Chapter Summary

| What You Need to Know | The Key Takeaway |
|----------------------|------------------|
| SAFEs | Not debt, not equity. A right to future shares. Use post-money SAFEs. |
| Valuation Cap | Sets the max valuation at which the SAFE converts. Protects early investors. |
| Convertible Notes | Like SAFEs but structured as debt (interest, maturity date). More leverage for investors. |
| Priced Rounds | Real equity issuance with preferred stock, term sheet, and full legal documentation. |
| Liquidation Preference | 1x non-participating is standard and fair. Resist participating preferred. |
| Anti-Dilution | Broad-based weighted average is standard. Reject full ratchet. |
| Board Composition | Maintain founder majority as long as possible. |
| Protective Provisions | Standard investor veto rights on existential decisions. Watch for overreach. |
| Option Pool | Created from pre-money valuation. Negotiate the size based on actual hiring needs. |
| Due Diligence | Prepare a data room. Have your legal house in order. This is where skipped paperwork haunts you. |
| Securities Law | You're selling securities. Exemptions exist (Reg D). File Form D. Don't advertise publicly. |

The instruments you use and the terms you agree to will define your economics for the life of the
company. Understand them before you sign.

---

**Next Chapter:** We'll pull back the curtain on the people behind the money — how VC funds actually
work internally, what drives angel investors, how accelerators pick winners, and what every type of
investor is really optimizing for when they negotiate with you.

---

*End of Chapter 3*
