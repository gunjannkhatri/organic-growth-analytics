# DAX Measures

All measures live in a dedicated table called `_Measures` (Enter Data → blank table → move measures into it).

**Model setup first**
- Mark `Dim_Date` as a date table (`Date` column).
- One active relationship from `Dim_Date[Date]` to each fact: `Fact_GSC[Date]`, `Fact_Web[Date]`, `Fact_Posts[PublishDate]`, `Fact_Followers[Date]`.
- `Fact_Targets[MonthStart]` → `Dim_Date[Date]` (targets sit on the 1st of each month, so use month-level visuals).
- Dimensions: `Dim_Query`, `Dim_Page`, `Dim_Device` → `Fact_GSC` · `Dim_Channel` → `Fact_Web` · `Dim_Platform`, `Dim_PostType`, `Dim_Pillar` → `Fact_Posts` · `Dim_Platform` → `Fact_Followers`.

---

## 1. Search (Google Search Console)

```dax
Impressions = SUM ( Fact_GSC[Impressions] )
```

```dax
Clicks = SUM ( Fact_GSC[Clicks] )
```

```dax
CTR = DIVIDE ( [Clicks], [Impressions] )
```

```dax
-- Impression-weighted average position (a simple AVERAGE of Position would be wrong)
Avg Position =
DIVIDE (
    SUMX ( Fact_GSC, Fact_GSC[Position] * Fact_GSC[Impressions] ),
    [Impressions]
)
```

```dax
Clicks MoM % =
VAR Prev = CALCULATE ( [Clicks], DATEADD ( Dim_Date[Date], -1, MONTH ) )
RETURN DIVIDE ( [Clicks] - Prev, Prev )
```

```dax
-- Negative = ranking improved
Position Change vs Last Month =
[Avg Position] - CALCULATE ( [Avg Position], DATEADD ( Dim_Date[Date], -1, MONTH ) )
```

## 2. Branded vs non-branded, device

```dax
Branded Clicks = CALCULATE ( [Clicks], Dim_Query[IsBranded] = TRUE () )
```

```dax
Non-Branded Clicks = CALCULATE ( [Clicks], Dim_Query[IsBranded] = FALSE () )
```

```dax
Non-Branded Click Share % = DIVIDE ( [Non-Branded Clicks], [Clicks] )
```

```dax
Mobile Click Share % =
DIVIDE ( CALCULATE ( [Clicks], Dim_Device[Device] = "Mobile" ), [Clicks] )
```

## 3. Opportunity: "striking distance" queries

Queries ranking on positions 8–20 get lots of impressions but few clicks. Moving them up is usually the quickest SEO win.

```dax
Striking Distance Queries =
COUNTROWS (
    FILTER (
        VALUES ( Dim_Query[Query] ),
        [Avg Position] >= 8 && [Avg Position] <= 20
    )
)
```

```dax
Striking Distance Impressions =
SUMX (
    FILTER (
        VALUES ( Dim_Query[Query] ),
        [Avg Position] >= 8 && [Avg Position] <= 20
    ),
    [Impressions]
)
```

```dax
Striking Distance Impression Share % =
DIVIDE ( [Striking Distance Impressions], CALCULATE ( [Impressions], ALLSELECTED ( Dim_Query ) ) )
```

## 4. Website traffic and leads (GA4-style)

```dax
Sessions = SUM ( Fact_Web[Sessions] )
```

```dax
Engaged Sessions = SUM ( Fact_Web[EngagedSessions] )
```

```dax
Engagement Rate = DIVIDE ( [Engaged Sessions], [Sessions] )
```

```dax
Web Leads = SUM ( Fact_Web[Leads] )
```

```dax
Lead Conversion Rate = DIVIDE ( [Web Leads], [Sessions] )
```

```dax
Organic Search Sessions =
CALCULATE ( [Sessions], Dim_Channel[Channel] = "Organic Search" )
```

```dax
Organic Search Leads =
CALCULATE ( [Web Leads], Dim_Channel[Channel] = "Organic Search" )
```

```dax
Organic Search Lead Share % =
DIVIDE ( [Organic Search Leads], CALCULATE ( [Web Leads], ALLSELECTED ( Dim_Channel ) ) )
```

```dax
Sessions Share % =
DIVIDE ( [Sessions], CALCULATE ( [Sessions], ALLSELECTED ( Dim_Channel ) ) )
```

## 5. Social media

```dax
Posts = COUNTROWS ( Fact_Posts )
```

```dax
Reach = SUM ( Fact_Posts[Reach] )
```

```dax
Post Impressions = SUM ( Fact_Posts[Impressions] )
```

```dax
Engagements = SUM ( Fact_Posts[Engagements] )
```

```dax
Engagement Rate (Social) = DIVIDE ( [Engagements], [Reach] )
```

```dax
Avg Reach per Post = DIVIDE ( [Reach], [Posts] )
```

```dax
Link Clicks = SUM ( Fact_Posts[LinkClicks] )
```

```dax
Link Click Rate = DIVIDE ( [Link Clicks], [Reach] )
```

```dax
Save + Share Rate =
DIVIDE ( SUM ( Fact_Posts[Saves] ) + SUM ( Fact_Posts[Shares] ), [Reach] )
```

```dax
-- Followers at the last date in the current selection
Followers =
VAR LastDate = MAX ( Fact_Followers[Date] )
RETURN CALCULATE ( SUM ( Fact_Followers[Followers] ), Fact_Followers[Date] = LastDate )
```

```dax
New Followers = SUM ( Fact_Followers[NewFollowers] )
```

```dax
Follower Growth % = DIVIDE ( [New Followers], [Followers] - [New Followers] )
```

## 6. Targets

```dax
Clicks Target = SUM ( Fact_Targets[OrganicClicksTarget] )
```

```dax
Clicks vs Target % = DIVIDE ( [Clicks], [Clicks Target] )
```

```dax
Organic Leads Target = SUM ( Fact_Targets[OrganicLeadsTarget] )
```

```dax
Organic Leads vs Target % = DIVIDE ( [Organic Search Leads], [Organic Leads Target] )
```

```dax
New Followers Target = SUM ( Fact_Targets[NewFollowersTarget] )
```

```dax
New Followers vs Target % = DIVIDE ( [New Followers], [New Followers Target] )
```

## 7. Dynamic titles (optional)

```dax
Selected Period Label =
FORMAT ( MIN ( Dim_Date[Date] ), "DD MMM YYYY" ) & " – " & FORMAT ( MAX ( Dim_Date[Date] ), "DD MMM YYYY" )
```
