# Organic Growth & SEO Analytics Platform

### Google Search Console + Website Traffic + Social Media → Power BI

An end-to-end organic marketing analytics solution: one star-schema model and a 3-page Power BI dashboard that shows how search, the website, and social media turn into visits and leads.

Companion projects: [marketing-performance-intelligence-platform](https://github.com/gunjannkhatri/marketing-performance-intelligence-platform) (paid ads, CPL, CAC) and [sales-pipeline-analytics](https://github.com/gunjannkhatri/sales-pipeline-analytics) (CRM pipeline and revenue).

---

## Business Problem

A small business publishes blogs and service pages, posts on several social platforms, and gets some leads from the website. But Search Console, website analytics, and each social platform live in separate dashboards, so nobody can see what is actually working.

This project answers:

- Is search traffic growing, and how much of it depends on the brand name?
- Which queries and pages are close to page one and worth improving?
- Which channels bring leads, not just visits?
- Which platforms, post formats, and content themes get the best engagement?
- Are we on track against monthly click, lead, and follower targets?

---

## Architecture

```
Search Console ─────┐
(queries, pages)    │
                    ▼
Website analytics ──► Power Query (M) ──► Star Schema ──► Power BI
(sessions, leads)   │   clean + type       5 Facts +       3-page dashboard
                    │                      9 Dims          scheduled refresh
Social platforms ───┘
(posts, followers)
```

In a real setup the CSV files are replaced by the Search Console API (or Windsor.ai), a GA4 export, and each platform's analytics export. Keys and columns stay the same, so the model and DAX don't change.

---

## Data Model (Star Schema)

### Dimension Tables

| Table | Purpose |
| --- | --- |
| Dim_Date | Shared calendar (day, week, month) |
| Dim_Query | Search query, intent, branded flag |
| Dim_Page | Page path, page type (Blog, Service, Home...), topic |
| Dim_Device | Mobile / Desktop |
| Dim_Channel | Website traffic channel (Organic Search, Direct, Social, Referral, Email, Paid) |
| Dim_Platform | Instagram, Facebook, LinkedIn, YouTube |
| Dim_PostType | Reel, Carousel, Image, Video, Link, Text, Short |
| Dim_Pillar | Content theme (Education, Product Showcase, Testimonial...) |

### Fact Tables

| Table | Grain |
| --- | --- |
| Fact_GSC | One row per day x query x device |
| Fact_Web | One row per day x channel |
| Fact_Posts | One row per social post |
| Fact_Followers | One row per day x platform |
| Fact_Targets | One row per month (clicks, leads, followers) |

---

## Key DAX Measures

| Measure | Logic |
| --- | --- |
| CTR | Clicks / Impressions |
| Avg Position | Impression-weighted average position |
| Non-Branded Click Share % | Clicks from non-brand queries / total clicks |
| Striking Distance Queries | Queries ranking on positions 8 to 20 |
| Lead Conversion Rate | Leads / Sessions, by channel |
| Organic Search Lead Share % | Organic Search leads / all web leads |
| Engagement Rate (Social) | Engagements / Reach |
| Avg Reach per Post | Reach / Posts |
| Followers | Followers at the last date in the selected range |
| Clicks vs Target % | Clicks / monthly target |

Full DAX code in `/dax/measures.md`

---

## Key Insights (from sample data)

**Search**
- Monthly clicks grew from about 3.3K (March) to about 8K (September) while average position improved from 11.3 to 9.0
- 84% of clicks come from non-branded queries, so growth is not just people searching the brand name
- Blog posts drive 59% of clicks; service pages drive 23%
- Service pages have plenty of impressions but weak CTR: /services/wall-painting gets 1.1% CTR on ~158K impressions and /projects gets 0.7% on ~109K
- 14 of 22 tracked queries rank on positions 8 to 20. They hold 64% of impressions but only 31% of clicks, so this is the biggest opportunity
- A two-week dip in late July cut non-branded clicks by about 43% (181 to 104 per day); it recovered to about 218 per day by mid-August
- Mobile is 68% of impressions but desktop has the higher CTR (3.8% vs 3.0%)

**Website and leads**
- Organic Search is 56% of sessions and 63% of leads, with a 3.2% lead conversion rate
- Organic Social is 10% of sessions but only 4% of leads (1.1% conversion), so it builds reach more than leads
- Email has the best conversion rate (4.2%) but only 3% of sessions

**Social**
- Instagram brings 66% of total reach with a 5.9% engagement rate
- Carousels get the best engagement (9.1% vs 5.1% for Reels), but Reels reach more people (about 6.1K vs 3.8K per post)
- On LinkedIn, carousels beat text posts (6.6% vs 3.6% engagement)
- Education and Behind the Scenes posts have the highest engagement (5.6% to 5.8%); Offer / Promo posts have the lowest (4.5%)
- Instagram posts at 7 to 9 PM get about 15% more reach than noon posts
- Total followers grew about 37% (8,900 to 12,170)

---

## Dashboard

### Page 1: Organic Overview

Clicks, impressions, CTR, position, organic leads, and followers, plus traffic by channel, leads by channel, and clicks vs target.

### Page 2: SEO Performance

Top queries, branded vs non-branded split, striking-distance opportunities, page-level CTR, and device split.

### Page 3: Social Media

Reach and engagement by platform, post format, and content theme, best posting hours, follower growth, and top posts.

Layout details in `/docs/dashboard_layout.md`

![Overview](docs/dashboard_overview.png)

![SEO](docs/dashboard_seo.png)

![Social](docs/dashboard_social.png)

---

## Tools Used

| Tool | Purpose |
| --- | --- |
| Power BI Desktop | Data modeling, DAX, dashboard |
| Power Query (M) | Data transformation |
| DAX | Calculated measures |
| Search Console / GA4 / platform exports | Source data (production) |
| CSV files | Source data (this demo) |

---

## Files

- `/data/` : Sample CSV files (synthetic data, not real company data)
- `/dax/measures.md` : All DAX measures
- `/powerquery/m_code.md` : Power Query M code for every table
- `/docs/` : Dashboard layout guide and screenshots

---

## How to Use

1. Clone the repo and open Power BI Desktop.
2. Create the `BaseUrl` parameter and `fnLoadCsv` function (see `/powerquery/m_code.md`).
3. Load the 5 fact and 9 dimension tables, create the relationships, and mark `Dim_Date` as a date table.
4. Paste the measures from `/dax/measures.md` into a `_Measures` table.
5. Build the three pages using `/docs/dashboard_layout.md`.

---

## Note on Data

All data in this repository is synthetically generated for demonstration purposes. The brand and website used in the sample ("BrightNest Interiors") are fictional. Numbers are realistic for a small business website but are not sourced from any real Search Console, analytics, or social account.

---

## Author

**Gunjan**, Data Analyst & Power BI Developer
[LinkedIn](https://www.linkedin.com/in/gunjan-khatri-00b1242ba) | [Portfolio](https://github.com/gunjannkhatri)
