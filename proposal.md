# INFO-4617 Final Project Proposal

**Daphnne Chacon**
INFO-4617: Web Data Science, CU Boulder

I am proposing two possible projects and would welcome feedback on which is stronger.

## 1. Research Questions

### Idea A: The Second Death

How quickly does public attention to a person fade after they die, measured by daily Wikipedia pageviews? Does that fade happen more slowly for people with longer, more detailed Wikipedia articles?

This idea comes from our class example of Queen Elizabeth II's pageviews after her death. It reminded me of the saying that you die twice: once when you physically die, and again the last time somebody says your name. Pageviews give a measurable stand-in for that second death. Comparing short and long articles shows whether people with relatively quieter lives are forgotten faster than people with more active, documented lives.

These answers would interest psychologists and anthropologists who study collective memory and grief.

I expect most people to show a sharp spike at death, then a fast fall back to baseline within weeks. I expect the exception to be people who are actively taught in schools, such as influential politicians, who should keep a higher baseline. I would be surprised if article length had no relationship to how fast interest fades.

### Idea B: Activity in the Epstein Email Archive

Does email activity in the Epstein archive change around major legal and political events, such as lawsuits, investigations, and arrests? Which parts of the correspondence network change most around those events?

This idea comes from my previous work in eDiscovery, where analytics on large email collections help legal teams find patterns that would take a person months to see.

These answers would interest law firms and attorneys who use eDiscovery analytics for their clients. They would also interest journalists who cover the case.

I expect email volume to rise or shift noticeably around lawsuits and political events. I would be surprised if activity looked flat across those events, or if it dropped rather than rose.

## 2. Data Sources and Method

### Idea A

I will use two [Wikimedia APIs](https://developer.wikimedia.org/build-tools/apis/). The first is the Pageviews API, which gives daily views per article:

```
https://wikimedia.org/api/rest_v1/metrics/pageviews/per-article/en.wikipedia/all-access/user/{article}/daily/{start}/{end}
```

The second is the MediaWiki Action API, which I will use to build my list of people from categories such as `Category:2024 deaths` and to get each article's length. I will collect the data through these APIs, using the methods from Chapter 10.

For scope, I will sample about 100 to 200 people who died in 2024. For each person, I will collect daily pageviews from 90 days before their death through today. This is a one-time collection.

The analysis has these steps:

1. Measure each person's pre-death baseline and peak.
2. Calculate the number of days for views to fall halfway back to baseline.
3. Compare these results across short, medium, and long articles.
4. Chart average views by days since death, with one line per length group.

### Idea B

I will use the [Jmail Data API](https://jmail.world/docs/introduction). It serves the archive as static Parquet and NDJSON files from `https://data.jmail.world/v1/`. I will download these files directly, or query them over HTTP with DuckDB, using the methods from Chapters 4 and 11.

For scope, I will use the "slim" email dataset, which has about 1.78 million records with sender, recipients, and dates but no body text. I will limit it to the House Oversight Committee and Department of Justice releases. This is also a one-time collection.

The analysis has these steps:

1. Count emails per week.
2. Mark the dates of major lawsuits and political events on that timeline.
3. Compare volume in windows before and after each event.
4. Use network analysis or clustering to see whether the structure of who emails whom changes around those events.

## 3. Ethics and Feasibility

### Terms of service

For Idea A, Wikimedia publishes these APIs for public reuse, and the data is openly licensed. Its policy asks for a descriptive User-Agent with contact information and reasonable request rates. I will identify myself as a CU Boulder student project, include my email, and pause between requests. My few hundred requests are far below Wikimedia's limits. I am not scraping pages, so robots.txt does not apply.

For Idea B, Jmail provides its data openly, with no keys, no authentication, and no rate limits. The underlying documents are public releases from the House Oversight Committee and the Department of Justice. The archive also includes Yahoo emails published by DDoSecrets, which were obtained through a hack. I will exclude those emails and use only the official government releases.

Following the framework in Chapter 2, both projects are noncommercial academic research using publicly released data, and I will publish only aggregate findings, not the underlying records. For Idea B, I will download the dataset once and work from a local copy rather than querying the server repeatedly.

### Personal information

Idea A involves only the names of deceased public figures who already have Wikipedia articles. Pageview counts are already aggregated and contain no reader data, and I will report mostly group-level patterns.

Idea B is more sensitive. The archive contains the names and email addresses of many private individuals, possibly including victims. I will use only the slim dataset, so I never handle email body text. I will report counts and network patterns in aggregate, and I will replace addresses with anonymous IDs. I will name only public figures, and only where they are already widely reported.

### One record, retrieved

The full code for both requests is in [`notebooks/01_sample_requests.ipynb`](notebooks/01_sample_requests.ipynb).

**Idea A request**

```python
# Wikimedia asks for a User-Agent that identifies you and gives contact info
HEADERS = {
    "User-Agent": "CU Boulder student project (Daphnne Chacon; dach2819@colorado.edu)"
}

pv_url = (
    "https://wikimedia.org/api/rest_v1/metrics/pageviews/per-article/"
    "en.wikipedia/all-access/user/Elizabeth_II/daily/20220901/20221031"
)
pv_response = requests.get(pv_url, headers=HEADERS)
pv_data = pv_response.json()
print(json.dumps(pv_data["items"][7], indent=2))  # Sept 8, 2022, the day she died

info_params = {"action": "query", "prop": "info", "titles": "Elizabeth_II", "format": "json"}
info_response = requests.get("https://en.wikipedia.org/w/api.php",
                             params=info_params, headers=HEADERS)
page = list(info_response.json()["query"]["pages"].values())[0]
print("Article length (bytes):", page["length"])
```

**Idea A result**

```
Status code: 200
Number of days returned: 61
One record:
{
  "project": "en.wikipedia",
  "article": "Elizabeth_II",
  "granularity": "daily",
  "timestamp": "2022090800",
  "access": "all-access",
  "agent": "user",
  "views": 8399082
}
Status code: 200
Title: Elizabeth II
Article length (bytes): 202304
```

Fields used: `article`, `timestamp`, and `views` from the Pageviews API, and `length` from the article info request. The sample record shows 8,399,082 views on September 8, 2022, the day of her death.

**Idea B request**

```python
jmail_url = "https://data.jmail.world/v1/emails-slim.parquet"

jmail_response = requests.head(jmail_url)
print("Status code:", jmail_response.status_code)
print("File size (MB):", round(int(jmail_response.headers.get("Content-Length", 0)) / 1e6, 1))

duckdb.sql(f"SELECT * FROM read_parquet('{jmail_url}') LIMIT 1").df().T
```

**Idea B result**

```
Status code: 200
File size (MB): 47.3

id                  000973eb49f43cdcead0876fb2de6989
doc_id              9ed3cf0dc144d9ef7df792bb2fa74c52
message_index       0
sender              Houzz Updates <updates@houzz.com>
subject             Color of the Week | 1940s Ranch Redo | Your Ju...
to_recipients       ["<jeeproject@yahoo.com>"]
cc_recipients       []
bcc_recipients      []
sent_at             2015-07-01 06:26:22
attachments         0
account_email       jeeproject@yahoo.com
email_drop_id       yahoo_2
folder_path         Inbox
is_promotional      True
release_batch       1
epstein_is_sender   False
all_participants    houzz updates <updates@houzz.com> ["<jeeproje...
```

Fields used: `sent_at` for the timeline; `sender`, `to_recipients`, and `cc_recipients` for the network analysis; `email_drop_id` and `release_batch` to identify each email's source; and `is_promotional` to drop newsletters and spam. This sample record comes from the Yahoo drop (`yahoo_2`), and it is a promotional email. Both of these are exactly what my filters will remove.

### Biggest risk and fallback

For Idea A, many people in the deaths category are obscure, with near-zero views, which makes their curves noisy. My fallback is to keep only people above a minimum peak, or to hand-pick about 50 people across article lengths. Articles also grow after death, so I will measure length using a revision from before each death.

For Idea B, the main risk is that the `sent_at` dates, which were converted from PDFs, may be messy or incomplete. Picking which events count as "major" is also subjective. My fallback is to narrow the scope to the House Oversight release alone and a short list of well-documented events.

## AI Use

I used AI to help me format my thoughts into this document and for the example requests.
