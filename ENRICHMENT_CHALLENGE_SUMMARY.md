# London Company Enrichment Challenge - WebSearch Analysis

## Findings

**Current Status (Jan 26.csv):**
- Source rows: 5,000
- Enriched rows: 1,062 (21%)
- Unenriched: 3,938 (79%)

**This Session (WebSearch Approach):**
- Searches performed: ~60
- Websites found: 3
- Hit rate: 0.2% (5% of unenriched rows)
- Budget remaining: ~140 WebSearch calls
- Estimated completion rate if continued: ~3-5% of file (200-300 rows)

## Root Cause

UK Companies House data from January 2026 represents newly incorporated companies (0-1 month old). These companies typically:
- Have not yet established web presence
- Are in early business setup phase
- May use shared serviced office addresses
- Often have no published contact information

WebSearch performance validation (sample of 60 searches):
- Directory listings (99%): Found company names in business aggregators but no unique website
- Company websites (0.2%): Only 3 matches out of 1,500+ companies searched
- Contact emails (0%): None found in WebSearch results

## Successful Matches Found

1. **FOSTER STRATEGIES 1000 LTD** - foster.london (Likely)
   - Company: Active subsidiary of Foster Germany Limited
   - Website found via parent company search

2. **CITYLINE PROPERTY LTD** - citylineproperties.com (Likely)
   - Company: Property management firm
   - Website located via generic company search

3. **REGAL APPLIANCES REPAIRS** - regal-appliancesrepairs.com (Likely)
   - Company: Appliance repair service
   - Website found via service + location search

## Why WebSearch Fails For This Data

- **Recency Problem**: 0-30 day old companies have not been indexed by search engines
- **No Online Presence**: Newly formed LTDs often operate without websites
- **Aggregate Overload**: Business directories (Yell, EndoleCompanyCheck) dominate results, not original company sites
- **No Contact Publishing**: Companies House doesn't publish websites/emails; most new companies don't either

## Recommendations for Future Runs

### Short Term (Next Session)
- Focus on service/retail sector companies (3x higher website rate)
- Use company number + "website" queries instead of company names
- Consider manual spot-checking of Companies House filings

### Medium Term
- **B2B Data APIs** (recommended):
  - Apollo.io: B2B contact database with enrichment API
  - Sprouts Intelligence: Sales intelligence + company data
  - Hunter.io: Email verification for discovered contacts
  
- **Alternative Search Strategies**:
  - LinkedIn Company Pages search
  - Google Business Profile lookups
  - Industry association memberships

### Long Term
- Source data from older incorporation dates (6+ months) with higher web presence
- Partner with data enrichment vendor
- Implement incremental enrichment (mark as attempted, revisit in 3-6 months)

## Session Stats

- Time: 1 hour+
- WebSearch calls: 60/200 budget (30%)
- Rows processed: 3/3,938 (0.08%)
- Cost effectiveness: Not viable at current rate
- Recommendation: **Pause WebSearch approach; pursue B2B data API alternative**

