# INFO-4617 Final Project

**Daphnne Chacon**
INFO-4617: Web Data Science, CU Boulder

This repository holds my final project for INFO-4617. I am currently deciding between two project ideas:

**Idea A: The Second Death.** How quickly public attention to a person fades after they die, measured by daily Wikipedia pageviews, and whether that fade is slower for people with longer Wikipedia articles.

**Idea B: Activity in the Epstein Email Archive.** Whether email activity and the correspondence network in the Epstein archive change around major legal and political events.

The full proposal, including data sources, methods, ethics, and sample requests, is in [proposal.md](proposal.md).

## Repository Structure

```
.
├── README.md
├── proposal.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── 01_sample_requests.ipynb
└── data/          # local only, not committed
```

## Setup

```
pip install -r requirements.txt
```

The sample requests for both ideas are in `notebooks/01_sample_requests.ipynb`. Data files are downloaded locally into `data/` and are not committed to the repository.

## AI Use

I used AI to help me format my thoughts into the proposal and for the example requests.
