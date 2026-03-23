# Product Compass Consulting

> Full-service AI product management consultancy — 48 agents, 65 skills, 8 departments covering the entire product lifecycle.

Built from [Pawel Huryn's PM Skills Marketplace](https://github.com/phuryn/pm-skills), which encodes proven PM frameworks from Teresa Torres, Marty Cagan, Alberto Savoia, Dan Olsen, and others into structured AI workflows.

## How it works

The **CPO** receives product challenges and routes them to the right department. Work flows through the organization following the product lifecycle:

```
Discovery -> Strategy -> Execution -> Launch
    ^                                    |
    <- <- Market Research <- <- <- <- <- <
              Data Analytics
```

| Phase | Department | What it does |
|-------|-----------|--------------|
| Discover | Product Discovery | Brainstorm ideas, map assumptions, design experiments, user research |
| Strategize | Product Strategy | Define vision, business models, competitive analysis, pricing |
| Build | Product Execution | PRDs, OKRs, roadmaps, sprints, stories, releases |
| Understand | Market Research | Personas, segmentation, journey maps, competitive landscape |
| Measure | Data Analytics | SQL queries, A/B test analysis, cohort retention |
| Launch | Go-to-Market | GTM strategy, beachhead segments, growth loops, battlecards |
| Grow | Marketing & Growth | Marketing campaigns, positioning, naming, North Star metrics |
| Support | PM Toolkit | Resume review, legal documents, proofreading |

## Org Chart

### Executive

| Agent | Title | Skills |
|-------|-------|--------|
| **CPO** | Chief Product Officer | paperclip |

### Product Discovery

| Agent | Title | Reports To | Skills |
|-------|-------|------------|--------|
| **VP of Product Discovery** | Vice President of Product Discovery | cpo | paperclip |
| Ideation Specialist | Ideation Specialist | vp-discovery | brainstorm-ideas-existing, brainstorm-ideas-new |
| OST Analyst | Opportunity Solution Tree Analyst | vp-discovery | opportunity-solution-tree |
| Assumption Analyst | Assumption Analyst | vp-discovery | identify-assumptions-existing, identify-assumptions-new, prioritize-assumptions |
| Experiment Designer | Experiment Designer | vp-discovery | brainstorm-experiments-existing, brainstorm-experiments-new |
| User Researcher | User Researcher | vp-discovery | interview-script, summarize-interview |
| Feature Analyst | Feature Analyst | vp-discovery | analyze-feature-requests, prioritize-features |
| Metrics Designer | Metrics Designer | vp-discovery | metrics-dashboard |

### Product Strategy

| Agent | Title | Reports To | Skills |
|-------|-------|------------|--------|
| **VP of Product Strategy** | Vice President of Product Strategy | cpo | paperclip |
| Vision Strategist | Vision Strategist | vp-strategy | product-strategy, product-vision |
| Business Model Analyst | Business Model Analyst | vp-strategy | lean-canvas, business-model, startup-canvas, value-proposition |
| Competitive Intel Analyst | Competitive Intelligence Analyst | vp-strategy | swot-analysis, pestle-analysis, porters-five-forces, ansoff-matrix |
| Pricing Strategist | Pricing Strategist | vp-strategy | pricing-strategy, monetization-strategy |

### Product Execution

| Agent | Title | Reports To | Skills |
|-------|-------|------------|--------|
| **VP of Product Execution** | Vice President of Product Execution | cpo | paperclip |
| PRD Writer | PRD Writer | vp-execution | create-prd |
| OKR Specialist | OKR Specialist | vp-execution | brainstorm-okrs |
| Roadmap Specialist | Roadmap Specialist | vp-execution | outcome-roadmap |
| Sprint Manager | Sprint Manager | vp-execution | sprint-plan, retro |
| Release Manager | Release Manager | vp-execution | release-notes |
| Risk Analyst | Risk Analyst | vp-execution | pre-mortem |
| Story Writer | Story Writer | vp-execution | user-stories, job-stories, wwas |
| Prioritization Specialist | Prioritization Specialist | vp-execution | prioritization-frameworks |
| QA Specialist | QA Specialist | vp-execution | test-scenarios |
| Stakeholder Analyst | Stakeholder Analyst | vp-execution | stakeholder-map |
| Meeting Analyst | Meeting Analyst | vp-execution | summarize-meeting |
| Data Generator | Data Generator | vp-execution | dummy-dataset |

### Market Research

| Agent | Title | Reports To | Skills |
|-------|-------|------------|--------|
| **Director of Market Research** | Director of Market Research | cpo | paperclip |
| Persona Specialist | Persona Specialist | director-market-research | user-personas, user-segmentation |
| Journey Mapper | Journey Mapper | director-market-research | customer-journey-map |
| Market Sizing Analyst | Market Sizing Analyst | director-market-research | market-sizing, market-segments |
| Competitive Analyst | Competitive Analyst | director-market-research | competitor-analysis |
| Sentiment Analyst | Sentiment Analyst | director-market-research | sentiment-analysis |

### Data Analytics

| Agent | Title | Reports To | Skills |
|-------|-------|------------|--------|
| **Director of Data Analytics** | Director of Data Analytics | cpo | paperclip |
| SQL Analyst | SQL Analyst | director-data-analytics | sql-queries |
| Experimentation Analyst | Experimentation Analyst | director-data-analytics | ab-test-analysis, cohort-analysis |

### Go-to-Market

| Agent | Title | Reports To | Skills |
|-------|-------|------------|--------|
| **Director of Go-to-Market** | Director of Go-to-Market | cpo | paperclip |
| GTM Strategist | GTM Strategist | director-gtm | gtm-strategy, beachhead-segment, ideal-customer-profile |
| Growth Strategist | Growth Strategist | director-gtm | growth-loops, gtm-motions |
| Battlecard Writer | Battlecard Writer | director-gtm | competitive-battlecard |

### Marketing & Growth

| Agent | Title | Reports To | Skills |
|-------|-------|------------|--------|
| **Director of Marketing** | Director of Marketing & Growth | cpo | paperclip |
| Marketing Strategist | Marketing Strategist | director-marketing | marketing-ideas, positioning-ideas, value-prop-statements |
| Brand Specialist | Brand Specialist | director-marketing | product-name |
| North Star Analyst | North Star Analyst | director-marketing | north-star-metric |

### PM Toolkit

| Agent | Title | Reports To | Skills |
|-------|-------|------------|--------|
| **Director of PM Toolkit** | Director of PM Toolkit | cpo | paperclip |
| Career Specialist | Career Specialist | director-toolkit | review-resume |
| Legal Specialist | Legal Document Specialist | director-toolkit | draft-nda, privacy-policy |
| Editor | Editor | director-toolkit | grammar-check |

## Skills (65)

All skills are referenced from [phuryn/pm-skills](https://github.com/phuryn/pm-skills) (MIT license, Pawel Huryn). Skills are not vendored — they point to the source repo.

| Plugin | Skills | Domain |
|--------|--------|--------|
| pm-product-discovery | 13 | Ideation, assumption testing, experiments, interviews |
| pm-product-strategy | 12 | Vision, business models, pricing, competitive analysis |
| pm-execution | 15 | PRDs, OKRs, roadmaps, sprints, release notes |
| pm-market-research | 7 | Personas, segmentation, journey maps, competitive intelligence |
| pm-data-analytics | 3 | SQL, cohort analysis, A/B testing |
| pm-go-to-market | 6 | GTM strategy, growth loops, battlecards |
| pm-marketing-growth | 5 | Marketing ideas, positioning, naming, North Star metrics |
| pm-toolkit | 4 | Resumes, legal documents, proofreading |

## Getting Started

Import this company into Paperclip:

```bash
paperclipai company import --from ./product-compass-consulting
```

## References

- **Source repo**: [phuryn/pm-skills](https://github.com/phuryn/pm-skills) — 65 skills and 36 chained workflows by Pawel Huryn
- **Agent Companies spec**: [agentcompanies.io/specification](https://agentcompanies.io/specification)
- **Paperclip**: [github.com/paperclipai/paperclip](https://github.com/paperclipai/paperclip)
- **The Product Compass**: [productcompass.pm](https://www.productcompass.pm)

## License

MIT — see [LICENSE](LICENSE)
