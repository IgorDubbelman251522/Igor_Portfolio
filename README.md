# Igor_Portfolio
(more content will go here)

# Year 1 Projects

(stuff I worked on in year 1 yadi yada)

# HivEye
 
A deep-learning app that finds a varroa mite hiding in a photo of a hundred bees, then tells the beekeeper what to do about it. HivEye reads a single photo of a colony, boxes every threat and health indicator it can see, and turns that into plain-language advice. It runs on a phone or a laptop, works offline, and is free. It started as a first-year university project, placed second in the Breda University of Applied Sciences Dragon's Den, and earned an invitation to an industry showcase where hiring companies, NVIDIA among them, were in the room. I am still building it.
 
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![YOLO26](https://img.shields.io/badge/Detector-YOLO26-orange)
![EfficientNetV2](https://img.shields.io/badge/Classifier-EfficientNetV2--S-green)
![PyTorch](https://img.shields.io/badge/PyTorch-CUDA%2012.4-red)
![SAHI](https://img.shields.io/badge/Inference-SAHI%20sliced-purple)
![ONNX](https://img.shields.io/badge/Export-ONNX%20%2F%20TensorRT-lightgrey)
 
## Demo
 
<!-- Add a GIF or screenshot: ![HivEye live detection](docs/demo.gif) -->
 
_A short clip of the live detector boxing mites on a colony photo goes here, plus the scan-to-advice flow._
 
The full codebase, with setup instructions, the model notebooks, and the run guide, lives in the project repository: [link to be added].
 
## Why this exists
 
Honeybees pollinate roughly three quarters of the world's food crops, and that pollination is worth something in the order of 153 billion euros a year. Across the EU, beekeepers lose twenty to thirty percent of their colonies annually, and in some regions more than half. The single biggest driver of that loss is a parasite called varroa destructor, a mite the size of a pinhead that latches onto bees and quietly collapses a hive from the inside.
 
The frustrating part is that varroa is treatable if you catch it early. The problem is catching it. The tools that exist are either a twenty to thirty minute manual inspection, which most hobbyists do too late and too rarely, or professional monitoring systems that cost between 500 and 2000 euros. There are more than 600,000 hobbyist beekeepers in the EU, most of them running between one and twenty-five hives, and almost none of them are served by either option.
 
HivEye is aimed squarely at that gap. A hobbyist invests something like 2400 euros to get a few colonies going, and one missed infestation can wipe the whole thing out. HivEye asks for nothing but a phone they already own. Point it at a frame of bees, and it tells you whether the thing that kills colonies is already in yours.
 
## The honest version of the story
 
I want to be straight about this project up front, because the way I learned to build it is the part I am most proud of, and it is the part a technical reader should care about most.
 
The first version of HivEye, the one that placed at Dragon's Den, reported 94.9 percent accuracy. That number was real, and it was also the most misleading thing about the project. It was measured on clean photos of a single bee, one subject per image. It answered the question "given a tidy picture of one thing, can you name the thing." That is not what the product does. The product has to find a small threat inside a crowded, messy, real photo, and when I stress-tested the original classifier on exactly that task, it failed six times out of six. The high headline number was being carried by the easy, over-represented classes, and it hid a near-total failure on the job that actually matters.
 
So I rebuilt the project around that failure instead of around the number. Everything below comes from treating "find the mite in the crowd" as the real problem and being honest about where the system is strong and where it is still thin. That reframe is what took HivEye from a good demo to something I could put in front of engineers and defend when they poked at it.
 
## How it works
 
The core insight is that this was never really an image-classification problem. Asking "is there a mite somewhere in this photo, and where" is an object-detection problem, and the research field solves it with object detectors rather than whole-image classifiers. The original approach slid a small window across the image and classified each tile, which was a slow, hand-rolled imitation of what a real detector does natively. I replaced it.
 
HivEye now runs on two models, each doing the job it is actually good at.
 
The detector is the headline. It is a YOLO26 model trained to draw boxes around individual varroa mites, wasps, pollen-bearing bees, and ordinary bees. To handle the fact that a mite might be fifteen pixels wide in a four-thousand-pixel photo, it uses sliced inference, cutting the full frame into overlapping tiles, running detection on each, and stitching the results back together. This is what makes "find the mite in the crowd" work in practice, and it is what drives the live overlay: point the camera at a whole frame and watch every threat get boxed and labelled in real time.
 
The classifier is the fast second opinion. It is an EfficientNetV2-S model, the direct descendant of the original Dragon's Den network, retrained on an enriched dataset. It gives a quick whole-image read of a close-up of a single subject, one bee or one mite, where you do not need localisation. In the app this is a separate "single" scan mode, distinct from the "crowd" mode that runs the detector.
 
Both models share the same four classes in the same fixed order everywhere, from training through to the app: bees, pollen-bearing bees, varroa, and wasps. Two of those are threats, varroa and wasps. Pollen-bearing bees are a positive signal, a sign the colony is foraging and healthy. Ordinary bees are the neutral baseline.
 
## The data
 
Data was the real bottleneck the first time around, so this cycle put the effort there first. I sourced seven public bounding-box datasets, verified every licence for commercial use, folded blocked and copyleft sets out of the product path entirely, and tracked the provenance of every image so I could reason about diversity later. The datasets were harmonised down to the four HivEye classes, with some careful judgment calls along the way, for example dropping a separate "mite" class that referred to a different parasite so it would not pollute the one class where recall matters most.
 
That merge produced a training set of just over 63,000 labelled boxes. The distribution is honest and uneven: varroa is abundant, with around 24,000 boxes, which is exactly the class I most want signal on. Bees are over-represented and were capped down using diversity-preserving selection rather than a blind random cut, because randomly throwing data away destroys the visual variety a model needs to generalise. That was a mistake I made in an earlier iteration and deliberately did not repeat. Wasps and pollen-bearing bees are the thin classes, and I have been candid throughout about them being the most likely weak spots.
 
Balancing by adding variety, not by deleting it, was the single most important data lesson of the whole project.
 
## Performance
 
The rule I hold myself to is that no headline number ever gets reported without the per-class breakdown beside it, because a global number is precisely what misled me the first time.
 
On a crowd-robustness stress test built from scenes graded by how densely the bees are packed, the detector caught all five of five varroa cases, including in the hardest layered-crowd images, and six of seven wasps. It also stayed calm on a clean control frame, raising no false alarm on a healthy hive, which matters just as much as catching threats. Its genuine weak spot is the pollen-bearing class, which traces directly back to limited training data for that class rather than to anything wrong with the architecture. Since a missed foraging bee is a cheap error and a missed mite is the one that kills a colony, this is the right place for the system to be weakest.
 
The classifier tells a clear before-and-after story. Retraining lifted its macro F1 across the four classes from 0.59 to 0.95, and the two classes that were weakest in the original Dragon's Den model, pollen and varroa, are the ones that improved most. I quote that as a clean-crop figure, because that is what it measures. Field performance on crowded photos is the detector's job.
 
I am equally clear about the caveat. These numbers come from synthetic and clean-crop test data. The genuinely honest, never-seen field number is the one piece of evidence I have not been able to close yet, and I would rather say that plainly than dress up a proxy as the real thing. Closing that gap is the first item on the roadmap below.
 
## The apps
 
HivEye ships as a working product, not a slide. There are two ways to run it, both built on the same model backend.
 
The desktop version runs a local Python server that loads the models and serves the app to a browser, with the detector and classifier each running in their own process because their underlying frameworks conflict in a single environment. A single launcher script sets everything up and starts it, and the two-process split is invisible to whoever is running it. Login and scan history are backed by a real database.
 
The mobile-style version is the same interface designed as a phone app, built around the live-detector overlay as the hero interaction: boxes and confidence labels drawn straight onto the camera feed. The scan flow is deliberately simple, capture or upload, then watch the results appear inline, then tap through for a per-class breakdown and a plain-language recommendation specific to what was found. A varroa detection tells you to treat within 48 hours and points you to your local association; a wasp detection tells you to reduce the hive entrance and monitor. There is an Expert Mode panel that highlights why the system flagged each detection, which is the hook for the explainability work.
 
The app also learned from watching real people use it. At Dragon's Den the jury could not work out how you would physically take the photo while wearing a bee suit, and in user testing people did not know whether to load the image or hit analyze first. So there is now a multi-chapter tutorial that teaches how to actually scan under real conditions, distance, angle, lighting, gloves on, on a frame versus at the entrance, along with what a good photo looks like versus a bad one.
 
## Accessibility
 
An app that is meant for every hobbyist beekeeper has to work for every hobbyist beekeeper, so accessibility is built in rather than bolted on. Every detection carries a visible confidence score, so nothing about the model's certainty is hidden from the user. Threats and positive indicators are colour-coded but never rely on colour alone; each result is labelled in text as well. The interface ships with three switchable themes, including a light high-contrast theme and a deep dark theme, so it stays legible in bright sun on a hive or in low light. Buttons carry text labels alongside their icons, a change that came directly out of user testing where icon-only controls confused people. The advice is written in plain language rather than jargon, and there are curated external resource links for anyone who wants to go deeper or reach a human expert.
 
## Tech stack
 
- **Python** across the whole system
- **YOLO26** (Ultralytics) for the object detector, the headline model
- **SAHI** for sliced inference, which is what solves the small-mite-in-a-huge-frame problem
- **EfficientNetV2-S** for the whole-image classifier
- **PyTorch** with CUDA 12.4 and automatic mixed precision for training
- **ONNX** for detector export, with a TensorRT path for fast inference and a TFLite path for the phone
- **SQLite** for the account and scan-history backend
- A self-contained **HTML, CSS, and JavaScript** front end served by a Python web server, no build step
## How it was trained
 
All training ran locally on a single laptop with an NVIDIA RTX 4060, with no cloud, using automatic mixed precision. The detector trained in a little over four hours, converged, and early-stopped rather than running the full schedule. Keeping the whole pipeline local and reproducible was a deliberate choice; it keeps costs controlled and keeps the system edge-minded, which is the same reason the deployment target is on-device inference rather than a server round-trip.
 
Along the way I kept honest notes on the acceleration story, including a moment where training silently ran on CPU because the installed framework build had no CUDA support. Catching that and fixing it took validation from around nine minutes an epoch down to about thirty seconds. It is a small anecdote, but it is the kind of thing that separates understanding your stack from reciting it.
 
## Roadmap
 
HivEye began as a Block C university project and I have long since been graded on it. I am continuing to build it anyway, as an off-university passion project, because I think it is worth finishing properly and because the problem it addresses has not gone away.
 
| Area | What I am working on next | Why it matters |
|------|---------------------------|----------------|
| Data depth | Grow the datasets for the classes that are currently thin, especially the pollen-bearing and wasp classes, so per-class recall rises where the data currently limits it | The one honest weak spot in the model traces to data volume, not architecture, so this is the highest-leverage improvement available |
| New classes | Introduce additional classes over time, both new threats and new positive health indicators, so HivEye reads more of what a beekeeper actually cares about | A richer read makes the product genuinely diagnostic rather than a single-threat checker |
| App features | Keep developing the app: a fully live account and history backend, a real notifying scan calendar for recurring inspections, and continued refinement of the scan flow and tutorial | Turns a one-shot tool into ongoing monitoring, which is what makes it something a beekeeper keeps using |
| Real-world validation | Build multiple honest, never-seen field test sets in collaboration with experienced beekeepers and industry contacts, and report per-class recall on them | This is the evidence gap I have been candid about, and closing it properly means working alongside people who know bees far better than I do |
 
## Background
 
I built HivEye in the first year of the ADS&AI programme at Breda University of Applied Sciences. It started as a coursework project spanning deep learning, responsible AI, and human-centered design, and it grew into the thing that got me into rooms with professionals: a second place in the university Dragon's Den, and an invitation to present at an industry showcase alongside hiring companies including NVIDIA. I did the model work, the data sourcing and licensing, the app, and the pitch. The part I value most is that I learned to find and admit my own system's blind spot, and to build honestly around it rather than around a flattering number. That is the engineer I am trying to be, and HivEye is where I learned to be one.


# Air Pollution and Wealth: A Power BI Dashboard
 
A Power BI dashboard that investigates whether a country's wealth is linked to how many of its people die from air pollution. It works through that question across seven pages, moving from a worldwide overview, to a close comparison of the most and least affected countries, to the factors sitting behind those numbers. The analysis is built on data from 2021 at the country level, held to a single year so the comparisons stay consistent.
 
![Power BI](https://img.shields.io/badge/Power%20BI-report-yellow)
![DAX](https://img.shields.io/badge/DAX-measures-blue)
![Power Query](https://img.shields.io/badge/Power%20Query-data%20prep-green)
![Azure Maps](https://img.shields.io/badge/Azure%20Maps-geospatial-lightgrey)
 
## Demo
 
<!-- Add screenshots: ![Dashboard overview](docs/overview.png) -->
 
_A few screenshots of the dashboard pages go here._
 
The full report, and the .pbix file to open in Power BI Desktop, is available here: [link to be added].
 
## What it explores
 
- A worldwide view of air pollution deaths in 2021, with an Azure Maps layer that sorts countries into danger categories and cards highlighting the highest and lowest totals.
- A focused set of subject countries for the comparison: China, India and Pakistan at the high end, Nauru, Palau and Greenland at the low end, with the United States and the Netherlands as familiar reference points.
- GDP per capita for those countries set against their death tolls, including a calculated figure for the correlation between wealth and air pollution deaths.
- Secondary angles that test the hypothesis: how national ageing relates to wealth, which age groups are hit hardest, and how the balance of indoor and outdoor deaths shifts between richer and poorer countries.
- A clear narrative from start to finish, opening with the research question and hypothesis and closing with the conclusions the data supports.
## Tech and data
 
Built in Power BI, with DAX for the aggregations and the correlation figure and Power Query for cleaning and aligning the source tables. The geographic views use the Azure Maps visual. The underlying data covers 2021 figures at the country level: deaths from air pollution, GDP per capita, an age breakdown of those deaths (from under 5 through to 70 and over), and a split between indoor and outdoor deaths. Every dataset was kept to 2021 so the comparisons line up in time.
 
_Data sources: add the providers you used (for example, WHO for pollution deaths and World Bank for GDP)._
 
## Background
 
I built this in the first block of my first year, as a data storytelling project tied to UN Sustainable Development Goal 3, Good Health and Well-being. I chose air pollution because it is an easy threat to overlook despite how much it affects both people and the climate, and I wanted the dashboard to serve as a starting point for identifying which countries carry the heaviest burden. My hypothesis was that a country's wealth would show up in the raw death counts and, more tellingly, in indirect signals such as how much of the population reaches old age and whether deaths happen indoors or outdoors. The later pages put that idea to the test.
 
# Tank Combat Simulation
 
An autonomous combat tank that navigates a 3D arena and destroys targets. Every action, from steering to aiming to firing, is learned through reinforcement learning rather than programmed by hand. The tank was modelled in Blender and runs inside a PyBullet physics simulation, where it learns to locate the red orbs, aim its turret, and fire.
 
![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![PyBullet](https://img.shields.io/badge/Physics-PyBullet-orange)
![Stable-Baselines3](https://img.shields.io/badge/RL-Stable--Baselines3-green)
![Gymnasium](https://img.shields.io/badge/API-Gymnasium-lightgrey)
 
## Demo
 
<!-- Add a GIF or screenshot: ![Tank demo](docs/demo.gif) -->
 
_Add a short clip or screenshot of the tank operating in the arena._
 
## Features
 
- A differential drive system with an independently rotating turret, so the tank can move and aim at the same time.
- Several arenas of varying layout and difficulty.
- Red orb targets to destroy and power-ups placed throughout each arena.
- A Gymnasium environment wrapping the observations, actions, and rewards, so the project integrates directly with Stable-Baselines3.
- Curriculum learning that starts with simpler arenas and progresses to harder ones for more stable training.
## Tech stack
 
- Python
- PyBullet (physics simulation and rendering)
- Stable-Baselines3 (reinforcement learning)
- Gymnasium (environment API)
- NumPy
- Blender (3D modelling)

## Background
 
I built this as an extracurricular challenge at the end of a coursework block, going beyond the required work to complete a full project on my own. It spans the entire pipeline, from modelling the tank in Blender, to simulating it in PyBullet, to training a reinforcement learning agent that can operate it. The aim was a system that has to perceive its surroundings, make decisions, and act, without any control logic written by hand.
 
# NGT Fingerspelling Recognition System
 
A recognition system for Dutch Sign Language (Nederlandse Gebarentaal) fingerspelling that works in real time. It reads your hand through a webcam and translates each sign into the corresponding letter. Rather than depending on an existing dataset, you record a few examples of each letter yourself and the system begins recognising them straight away, tracking 21 points on the hand with MediaPipe.
 
![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![MediaPipe](https://img.shields.io/badge/MediaPipe-hand%20tracking-orange)
![OpenCV](https://img.shields.io/badge/OpenCV-computer%20vision-green)
![License](https://img.shields.io/badge/License-MIT-lightgrey)
 
## Demo
 
<!-- Add a GIF or screenshot: ![Sign recognition demo](docs/demo.gif) -->
 
_A short clip of the system recognising fingerspelling from the webcam goes here._
 
The full project, including setup instructions and technical details, is available here: [link to be added].
 
## Features
 
- Works without a training dataset. You record two or three examples of each letter and the system can recognise them immediately.
- Recognises signs in real time from a live webcam feed.
- Supports both hands. You can record with one hand and recognise with either, since left hand input is mirrored automatically.
- Runs entirely in the browser, with the 21 tracked hand points drawn over the live video.
- Uses a voting system across recent frames to keep predictions stable, so the detected letter holds steady instead of flickering between similar signs.
## Tech stack
 
- Python
- MediaPipe (hand landmark tracking)
- OpenCV (webcam and image handling)
- NumPy (numerical computation)
- A browser front end (HTML, CSS, JavaScript) served by a Python web server

## Background
 
We built this for the second end of block challenge in the ADS&AI programme. The brief was to recognise sign language without an existing dataset to train on, so the system lets each user record their own reference signs before recognition begins. Most of our effort went into making it reliable in real conditions, where changes in lighting, hand position, and signs that look alike all make recognition harder.

# RegimeIQ
 
A risk-personalised, regime-aware investment advisory system for the Nasdaq-100. RegimeIQ reads the state of the wider market, sorts every stock in the index by how risky it currently is, weighs up news sentiment and price technicals, and combines all of that into a single BUY or HOLD signal for each ticker. It then explains why. The recommendations are filtered to match the risk appetite you set during onboarding, and a portfolio layer suggests how much to put into each position. Nothing is executed on your behalf. Every output is advisory, and you make the final call.
 
The system is built from four machine learning models that feed into one another, a Streamlit dashboard, a SHAP explainability layer, and a Claude-powered narrator that turns model output into plain language.
 
![Python](https://img.shields.io/badge/Python-3.11%2B-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-clustering-orange)
![XGBoost](https://img.shields.io/badge/XGBoost-gradient%20boosting-green)
![FinBERT](https://img.shields.io/badge/FinBERT-sentiment-red)
![SHAP](https://img.shields.io/badge/SHAP-explainability-purple)
![Streamlit](https://img.shields.io/badge/Streamlit-dashboard-ff4b4b)
![Docker](https://img.shields.io/badge/Docker-containerised-blue)
![Tests](https://img.shields.io/badge/tests-[COUNT]%20passing-brightgreen)
![Coverage](https://img.shields.io/badge/coverage-[XX]%25-brightgreen)
 
## Demo
 
<!-- Add screenshots or a GIF: ![Recommendations page](docs/recommendations.png) -->
 
_A short walkthrough of the dashboard goes here: onboarding, the daily briefing, and a recommendation with its SHAP breakdown._
 
The full codebase, with setup instructions, the SQLite schema, and the model iteration notebooks, lives in the project repository: [link to be added].
 
## The idea
 
Most retail investors either follow tips with no reasoning behind them or freeze up in front of research reports they do not have time to read. RegimeIQ sits in between. It gives you a data-grounded second opinion on Nasdaq-100 stocks: what the market is doing right now, which stocks fit your risk level, and where the model sees an edge, with the reasoning attached to every call.
 
The core bet is that a signal means different things in different market conditions. A bullish technical setup during a calm bull market is not the same as the identical setup in a high-volatility sell-off. So instead of one model looking at stocks in isolation, RegimeIQ layers market context and per-stock risk underneath the signal, then lets a final model learn how to weigh those pieces together.
 
## How it fits together
 
Historical data (Nasdaq-100 price and volume, macroeconomic indicators, and a cache of news headlines) flows through a set of ETL pipelines into a shared SQLite database. Each model reads from that database, produces its predictions, and writes them back to its own output table. The dashboard reads only from the database, so no model runs at request time and pages load quickly.
 
The four models are arranged in a chain. Two of them describe the environment (what regime the market is in, and how risky each stock is). One reads sentiment and technicals. The fourth combines everything into the final recommendation.
 
```
Historical CSVs (OHLCV, macro indicators, news cache)
       |
  ETL pipelines  ->  SQLite database
       |
  Models (trained once, loaded for inference)
    - Regime Detector      -> market regime
    - Risk-Tier Classifier -> per-stock risk tier
    - Sentiment Classifier  -> per-ticker signal + SHAP
    - Meta-Learner          -> final ensemble BUY/HOLD
       |
  Streamlit dashboard (5 pages)
```
 
## The models
 
### Regime Detector
 
This model answers a single question: what kind of market are we in today? It clusters standardised macroeconomic features into four states, labelled Bull, Bear, High Volatility, and Sideways. The features include the VIX, the Fed funds rate, CPI, the 10-year Treasury yield, GDP growth, gold and oil prices, and global index momentum.
 
The approach is KMeans with k=4, with a Hidden Markov Model as a fallback when regimes flip too quickly to be useful. A regime that changes every other day is noise rather than signal, so the model is tuned for persistence. The average time spent in a regime has to clear a minimum dwell time, and two people labelling the same periods by hand have to agree at least [XX]% of the time before the labelling is trusted. Achieved mean dwell time was [XX] days at [XX]% rater consistency.
 
The daily regime label feeds every downstream model and drives the plain-language market summary on the dashboard.
 
### Risk-Tier Classifier
 
This model sorts each stock in the index into Low, Medium, or High risk, refreshed monthly. It clusters per-stock risk features: 60-day rolling annualised volatility, 12-month maximum drawdown, beta against the Nasdaq-100 index, and sentiment volatility drawn from the sentiment pipeline.
 
It uses KMeans with k=3, with a Gaussian Mixture Model as the fallback. Two things matter for a risk label to be worth showing. It has to carve the stocks into genuinely distinct groups, measured by silhouette score, and it has to be stable, because a stock that jumps from Low to High and back month to month is not describing real risk. The target was a silhouette score above 0.35 with at least 80% of tickers holding their tier month to month; the model reached [XX] and [XX]%.
 
The risk tier is what personalises the whole system. Your onboarding answers map you to one of these tiers, and the recommendations you see are filtered to match.
 
### Sentiment Signal Classifier
 
This model produces a per-ticker probability that a stock will rise over the coming week. It is an XGBoost classifier trained on a mix of technical indicators and news sentiment: RSI, MACD (line, signal, and histogram), moving-average ratios across the 20, 50, and 200-day windows, daily return statistics, the current regime label, and sentiment features.
 
The sentiment features come from FinBERT, a financial-domain language model that scores news headlines. Each headline gets a sentiment score and a confidence value, and the pipeline also tracks how volatile sentiment has been, on the reasoning that steady positive coverage and wildly swinging coverage should not be treated the same.
 
Every prediction ships with SHAP values, so the exact contribution of each feature to that specific probability is stored alongside the score rather than hidden inside the model. Walk-forward F1 on the held-out final year was [XX] against a target of 0.60.
 
### Risk-Aware Classifier and Stacking Meta-Learner
 
The final stage is two models. The risk-aware classifier is a second XGBoost model that deliberately ignores sentiment and instead leans on technicals plus the regime label and the risk tier. Running it alongside the sentiment model gives two independent readings of the same stock from different angles.
 
The meta-learner then combines them. It is a stacking model, trained only on out-of-fold predictions from the two branches so it never sees the answers to the examples it learns from, which is what keeps a stacked ensemble honest. It started as logistic regression and moved to a shallow XGBoost. The output is the final BUY or HOLD probability that the dashboard ranks stocks by.
 
The real test of an ensemble is whether it actually beats its best single component. The bar was set in advance. The ensemble had to add at least two points of F1 over the strongest individual branch, and if it did not, the plan was to fall back to that single branch and say so plainly. The ensemble reached [XX] F1 with a lift of [XX] points, and BUY precision of [XX].
 
## The application
 
The dashboard is a five-page Streamlit app.
 
**Onboarding.** A short wizard collects your budget, your risk tolerance, and your investment horizon, then maps those answers onto one of the three risk tiers. This is the step that personalises everything else. It also collects consent up front and explains, in plain language, what the system stores and what it does not.
 
**Daily Briefing.** Shows the current market regime with a paragraph, generated through the Claude API, explaining what that regime means and how to read it. This is where the market-context model surfaces for the user.
 
**Recommendations.** The core page. Stocks are sorted by the ensemble model's BUY probability and filtered to your risk tier, so a conservative user never sees a high-risk stock pushed to the top. Each recommendation carries a SHAP chart showing which features drove it, and the Markowitz allocation layer suggests position sizes across the shortlist. The sizes are suggestions. You can ignore or change any of them.
 
**Backtest.** Shows how the system performed under walk-forward evaluation, where the models are only ever tested on data from after the period they were trained on, plus a dedicated stress test over the Q4 2022 drawdown to see how the recommendations hold up when the market turns.
 
**Chat.** A conversational page, running on the Claude API and grounded in the live dashboard data, where you can ask follow-up questions about a recommendation before acting on it. It doubles as a check on the system: if a call looks off, you can interrogate it rather than take it at face value.
 
## Explainability and oversight
 
Two things were treated as requirements rather than extras. The first is that no recommendation is a black box. SHAP attributions are computed and stored per prediction and surfaced on the Recommendations page, so a BUY is always accompanied by the reasons for it. The second is that a human stays in the loop. An operator override allows recommendations to be suppressed or downgraded before they reach a user, every recommendation is framed as advisory, and the system never executes a trade. These choices also map onto the transparency and human-oversight obligations in the EU AI Act, which the project documents alongside its GDPR position.
 
## Tech stack
 
- **Python** across the whole system
- **SQLite** as the shared datastore, with idempotent ETL pipelines that produce the same state on every run
- **scikit-learn** for the clustering models (KMeans, GMM) and the logistic baseline
- **XGBoost** for the signal classifiers and the stacked meta-learner
- **FinBERT** (ProsusAI/finbert) for news headline sentiment
- **SHAP** for per-prediction feature attribution
- **Streamlit** for the dashboard
- **Claude API** for the market narrator and the chat page
- **Docker** and docker-compose for a reproducible deployment
- **pytest** for the test suite, at [XX]% coverage across [COUNT] tests
## Data
 
The system runs on historical data provided for the project: Nasdaq-100 OHLCV price history, a set of macroeconomic indicators (VIX, Fed funds rate, CPI, Treasury yields, GDP growth, commodities), and a cache of news headlines scored for sentiment. All models are evaluated with walk-forward validation, holding out the most recent year so the reported numbers reflect performance on data the models never trained on.
 
## Background
 
RegimeIQ was built by a team of four for the Block D capstone in the first year of the ADS&AI programme at Breda University of Applied Sciences. The brief was to design and build an end-to-end machine learning application, from data pipeline through to a working dashboard, along with the evaluation, testing, and compliance documentation that would sit behind a real product. I was responsible for the risk-tier classifier and its pipeline, and worked across the other models, the ETL layer, and the application as the system came together.
 


