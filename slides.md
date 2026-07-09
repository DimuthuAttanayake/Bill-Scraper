SLIDE 1

Who's Buying and Selling Their Own Stock?

A searchable record of corporate insider trades, built from SEC filings and daily stock prices.

Dimuthu Attanayake
Data and Databases, July 9


SLIDE 2

The Topic

Company directors, officers, and big shareholders have to file a Form 4 with the SEC within two business days of buying or selling their own company's stock.

It is one of the few public windows into what the people who know a company best are doing with their money.

The filings are public, but they are scattered and full of jargon, so a normal reader can't really use them.

My question: who is trading their own stock right now, are they buying or selling, and are they trading above or below the market price?


SLIDE 3

The Data: Two Sources

Source 1: SEC EDGAR (scraped)
The "latest filings" feed of Form 4 disclosures. This gives me the insider, the company, the role, and the actual trades (shares, price, buy or sell).

Source 2: Yahoo Finance (API)
The daily closing price for each company. This lets me answer the part Form 4 can't: did the insider trade above or below what the public was paying that day?

I chose them because the filing tells me what happened, and the price gives the context that makes one trade meaningful.


SLIDE 4

The Process

I scrape the latest 100 Form 4 filings, then follow each one to its XML to get the full trade details and footnote text.

Each filing gets a hash, so I can tell new filings from ones I already have and skip re-scraping. Every run writes a change log and an error log.

A GitHub Action runs the scrape every day and saves the new filings, so the dataset grows on its own. It went from 100 to about 300 rows in three days.

Then I pull the stock prices for each ticker and join them to the trades.


SLIDE 5

Challenges So Far

The same filing shows up twice, once under the insider and once under the company, so 100 rows was really 42 filings. I fix this by deduping on the accession number.

The real data is nested in XML and has a different shape for different trade types, so flattening it into clean rows took care.

Prices don't always match. Weekend trades have no closing price and small or foreign tickers aren't on Yahoo, so about 10% of trades get no price.

A scoping question: Form 4 is a new dataset, so I am using the stock prices as a second source to meet the requirement.


SLIDE 6

The Schema

Two tables, joined together. One filing can have many trades.

filings table (one row per filing): accession, insider name, company, ticker, relationship, filing date, footnotes, SEC link.

trades table (one row per transaction): accession, transaction date, code (buy or sell), shares, price, value, market close, percent vs market.

The two price columns are where the second source joins in, matched on ticker and transaction date.

One row per transaction is the searchable unit, because that is how a reader thinks: "show me the buys."


SLIDE 7

The Public App

A hosted page built with Flask and SQLite, with a search bar, pagination, and a CSV export.

Users can search by insider, company, ticker, or role, and filter by buy vs sell.

Example search 1: every open-market purchase by a director. Selling is routine, but an insider buying with their own cash is the rare, interesting event.

Example search 2: all trades for one ticker, showing whether each was above or below market.

(Show wireframe sketch here.)


SLIDE 8

Methodology, Horizon, and Questions

What it reveals: a near real-time, verifiable record of insider trades. Every row links back to the original SEC filing.

What is missing: recent filings only, no long history. Most activity is selling and grants; open-market buys are rare. It shows what was traded, not why.

Ethics: this is public data, but it aggregates named people's finances. A trade is not proof of intent, so the framing should not imply wrongdoing.

Where it goes next: ship the app on Render, finalize the tables, add filters, write the full methodology.

Questions for you:
1. Is one row per transaction the clearest unit for a non-expert, or should I group by filing or by insider?
2. How do I show "traded 2% below market" as useful context without implying a signal that isn't there?
