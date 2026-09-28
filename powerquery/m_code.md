# Power Query (M) Code

Create a parameter called `BaseUrl` first (Home → Manage Parameters → New, type Text):

- **From GitHub:** `https://raw.githubusercontent.com/gunjannkhatri/organic-growth-analytics/main/data/`
- **From a local folder:** `C:\path\to\organic-growth-analytics\data\` and use `File.Contents(BaseUrl & FileName)` instead of `Web.Contents` in `fnLoadCsv`

---

## Helper function: fnLoadCsv

Create a blank query named `fnLoadCsv` and paste this. Every table below calls it, so the load logic lives in one place.

```m
(FileName as text) as table =>
let
    Source = Csv.Document(
        Web.Contents(BaseUrl & FileName),
        [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
    ),
    Promoted = Table.PromoteHeaders(Source, [PromoteAllScalars = true])
in
    Promoted
```

---

## Fact tables

```m
// Fact_GSC
let
    Source = fnLoadCsv("fact_gsc.csv"),
    Typed = Table.TransformColumnTypes(Source, {
        {"Date", type date}, {"QueryKey", Int64.Type}, {"PageKey", Int64.Type}, {"DeviceKey", Int64.Type},
        {"Impressions", Int64.Type}, {"Clicks", Int64.Type}, {"Position", type number}
    }, "en-US")
in
    Typed
```

```m
// Fact_Web
let
    Source = fnLoadCsv("fact_web.csv"),
    Typed = Table.TransformColumnTypes(Source, {
        {"Date", type date}, {"ChannelKey", Int64.Type}, {"Sessions", Int64.Type},
        {"NewUsers", Int64.Type}, {"EngagedSessions", Int64.Type}, {"Leads", Int64.Type}
    }, "en-US")
in
    Typed
```

```m
// Fact_Posts
let
    Source = fnLoadCsv("fact_posts.csv"),
    Typed = Table.TransformColumnTypes(Source, {
        {"PostID", type text}, {"PublishDate", type date}, {"PublishHour", Int64.Type},
        {"PlatformKey", Int64.Type}, {"PostTypeKey", Int64.Type}, {"PillarKey", Int64.Type},
        {"Reach", Int64.Type}, {"Impressions", Int64.Type}, {"Likes", Int64.Type},
        {"Comments", Int64.Type}, {"Shares", Int64.Type}, {"Saves", Int64.Type},
        {"Engagements", Int64.Type}, {"LinkClicks", Int64.Type}
    }, "en-US")
in
    Typed
```

```m
// Fact_Followers
let
    Source = fnLoadCsv("fact_followers.csv"),
    Typed = Table.TransformColumnTypes(Source, {
        {"Date", type date}, {"PlatformKey", Int64.Type}, {"Followers", Int64.Type}, {"NewFollowers", Int64.Type}
    }, "en-US")
in
    Typed
```

```m
// Fact_Targets
let
    Source = fnLoadCsv("fact_targets.csv"),
    Typed = Table.TransformColumnTypes(Source, {
        {"MonthStart", type date}, {"OrganicClicksTarget", Int64.Type},
        {"OrganicLeadsTarget", Int64.Type}, {"NewFollowersTarget", Int64.Type}
    }, "en-US")
in
    Typed
```

---

## Dimension tables

```m
// Dim_Date
let
    Source = fnLoadCsv("dim_date.csv"),
    Typed = Table.TransformColumnTypes(Source, {
        {"Date", type date}, {"Year", Int64.Type}, {"MonthNum", Int64.Type}, {"Month", type text},
        {"YearMonth", type text}, {"MonthStart", type date}, {"WeekStart", type date},
        {"Weekday", type text}, {"IsWeekend", type logical}
    }, "en-US")
in
    Typed
```

```m
// Dim_Query
let
    Source = fnLoadCsv("dim_query.csv"),
    Typed = Table.TransformColumnTypes(Source, {
        {"QueryKey", Int64.Type}, {"Query", type text}, {"Intent", type text}, {"IsBranded", type logical}
    })
in
    Typed
```

```m
// Dim_Page
let
    Source = fnLoadCsv("dim_page.csv"),
    Typed = Table.TransformColumnTypes(Source, {
        {"PageKey", Int64.Type}, {"PagePath", type text}, {"PageType", type text}, {"Topic", type text}
    })
in
    Typed
```

```m
// Dim_Device
let
    Source = fnLoadCsv("dim_device.csv"),
    Typed = Table.TransformColumnTypes(Source, {{"DeviceKey", Int64.Type}, {"Device", type text}})
in
    Typed
```

```m
// Dim_Channel
let
    Source = fnLoadCsv("dim_channel.csv"),
    Typed = Table.TransformColumnTypes(Source, {{"ChannelKey", Int64.Type}, {"Channel", type text}})
in
    Typed
```

```m
// Dim_Platform
let
    Source = fnLoadCsv("dim_platform.csv"),
    Typed = Table.TransformColumnTypes(Source, {{"PlatformKey", Int64.Type}, {"Platform", type text}})
in
    Typed
```

```m
// Dim_PostType
let
    Source = fnLoadCsv("dim_post_type.csv"),
    Typed = Table.TransformColumnTypes(Source, {{"PostTypeKey", Int64.Type}, {"PostType", type text}})
in
    Typed
```

```m
// Dim_Pillar
let
    Source = fnLoadCsv("dim_pillar.csv"),
    Typed = Table.TransformColumnTypes(Source, {{"PillarKey", Int64.Type}, {"ContentPillar", type text}})
in
    Typed
```

---

## Notes

- Set `fnLoadCsv` and `BaseUrl` to **Connection only** (right-click → uncheck Enable load).
- Data privacy: if Power BI warns about combining sources, set the privacy level of the GitHub source to Public.
- In production, replace the CSVs with real exports: Search Console API (or Windsor.ai), GA4 export to BigQuery, and Meta / LinkedIn / YouTube analytics. Keys and column names stay the same, so the model and DAX don't change.
