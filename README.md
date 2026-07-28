# Igor Dubbelman
 
<img src="https://igordubbelman251522.github.io/Igor_Portfolio/images/profile.jpeg" width="150" style="border-radius: 50%; float: right; margin-left: 30px;">

Second-year Data Science & AI student at Breda University of Applied Sciences, building practical machine learning and computer vision systems. I care most about doing the honest version of the work: finding where a model actually breaks and building around it rather than around a flattering number.
 
 
Below are the projects I'm most proud of so far. Each links to its own repository with the full write-up, the code, and instructions to run it yourself.
 
---

# Year 1 Projects

### All of these projects were built during my first year, where I earned a 3.67 GPA. Two of them opened doors beyond the classroom: I was selected among the top 5 students out of more than 100 to present at the Breda University of Applied Sciences Dragon's Den, where HivEye placed second in front of a panel of investors, and again among the top 5 to present at the university's first industry showcase, with hiring companies in the room, NVIDIA among them. 

---
 
## [HivEye](https://github.com/IgorDubbelman251522/HivEye)
 
![](/images/hiveye_logo_2.png)
 
A deep-learning app that finds a single varroa mite hiding in a photo of a hundred bees, then tells the beekeeper what to do about it. Point a phone at a frame of bees and it boxes every threat and health indicator it sees, then turns that into plain-language advice. It runs offline, on hardware people already own, and it's free.
 
It started as a first-year university project, placed second in the Breda University of Applied Sciences Dragon's Den, and earned an invitation to an industry showcase with hiring companies in the room, NVIDIA among them. I'm still building it.
 
The part I'm proudest of isn't the accuracy number, it's that I caught my own blind spot: the first version reported 94.9% accuracy and then failed six times out of six on the task that actually matters, finding a small threat in a crowded real photo. I rebuilt the whole project around that failure rather than around the flattering number.
 
![Python](https://img.shields.io/badge/Python-3.10%2B-blue) ![YOLO26](https://img.shields.io/badge/Detector-YOLO26-orange) ![EfficientNetV2](https://img.shields.io/badge/Classifier-EfficientNetV2--S-green) ![PyTorch](https://img.shields.io/badge/PyTorch-CUDA%2012.4-red)
 
**Key concepts:** object detection, image classification, small-object detection, sliced inference, transfer learning, per-class recall, edge / on-device inference
 
![](/images/HivEye.gif)
 
---
 
## [RegimeIQ](https://github.com/BredaUniversityADSAI/2025-26d-fai1-adsai-group_6_2026)
 
A risk-personalised, regime-aware investment advisory system for the Nasdaq-100. It reads what the wider market is doing, sorts every stock in the index by how risky it currently is, weighs news sentiment against price technicals, and turns all of it into a single BUY or HOLD call per stock, with the reasoning attached. Everything is advisory; it never trades for you.
 
Under the hood it's four machine-learning models feeding into one another, a Streamlit dashboard, a SHAP layer so no recommendation is a black box, and a Claude-powered narrator that explains the market in plain language. Built by a team of four for our first-year capstone; I owned the risk-tier classifier and its pipeline, and worked across the rest.
 
![Python](https://img.shields.io/badge/Python-3.11%2B-blue) ![XGBoost](https://img.shields.io/badge/XGBoost-gradient%20boosting-green) ![FinBERT](https://img.shields.io/badge/FinBERT-sentiment-red) ![SHAP](https://img.shields.io/badge/SHAP-explainability-purple) ![Streamlit](https://img.shields.io/badge/Streamlit-dashboard-ff4b4b) ![Docker](https://img.shields.io/badge/Docker-containerised-blue)
 
**Key concepts:** clustering, classification, ensemble learning, model stacking, feature engineering, time-series features, walk-forward validation, sentiment analysis, NLP, explainability (SHAP), portfolio optimisation
 
<img src="https://igordubbelman251522.github.io/Igor_Portfolio/images/regime1.png" width="250"> <img src="https://igordubbelman251522.github.io/Igor_Portfolio/images/regime2.png" width="250"> <img src="https://igordubbelman251522.github.io/Igor_Portfolio/images/regime3.png" width="250">
 
---
 
## [Air Pollution and Wealth: A Power BI Dashboard](https://github.com/IgorDubbelman251522/Dashboard_Igor_Y1A)
 
A Power BI dashboard that asks whether a country's wealth is linked to how many of its people die from air pollution. It works through that question across seven pages, from a worldwide map down to the factors sitting behind the numbers, and puts a hard figure on the correlation between wealth and pollution deaths.
 
I built it for a data-storytelling brief tied to UN Sustainable Development Goal 3, using 2021 country-level data held to a single year so the comparisons stay honest.
 
![Power BI](https://img.shields.io/badge/Power%20BI-report-yellow) ![DAX](https://img.shields.io/badge/DAX-measures-blue) ![Power Query](https://img.shields.io/badge/Power%20Query-data%20prep-green) ![Azure Maps](https://img.shields.io/badge/Azure%20Maps-geospatial-lightgrey)
 
**Key concepts:** data visualisation, data storytelling, exploratory data analysis, correlation analysis, geospatial analysis, data cleaning, business intelligence
 
<img src="https://igordubbelman251522.github.io/Igor_Portfolio/images/power1.png" width="250"> <img src="https://igordubbelman251522.github.io/Igor_Portfolio/images/power2.png" width="250"> <img src="https://igordubbelman251522.github.io/Igor_Portfolio/images/power3.png" width="250">
 
---
 
## [Tank Combat Simulation](https://github.com/BredaUniversityADSAI/combat_robotics/tree/Team-3)
 
An autonomous combat tank that learns to navigate a 3D arena and destroy targets. Every action, steering, aiming, firing, is learned through reinforcement learning rather than programmed by hand. I modelled the tank in Blender and trained it inside a PyBullet physics simulation, starting on simple arenas and working up to harder ones.
 
I built this as an extracurricular challenge at the end of a coursework block, going past the required work to take one system all the way from a 3D model to a trained agent that perceives, decides, and acts on its own.
 
![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![PyBullet](https://img.shields.io/badge/Physics-PyBullet-orange) ![Stable-Baselines3](https://img.shields.io/badge/RL-Stable--Baselines3-green) ![Gymnasium](https://img.shields.io/badge/API-Gymnasium-lightgrey)
 
**Key concepts:** reinforcement learning, deep reinforcement learning, reward shaping, curriculum learning, physics simulation, autonomous agents, 3D modelling
 
![](/images/tank.gif)
 
---
 
## [NGT Fingerspelling Recognition System](https://github.com/BredaUniversityADSAI/Group-3-Sign-Language-)
 
A real-time recognition system for Dutch Sign Language (NGT) fingerspelling. It reads your hand through a webcam and translates each sign into its letter. The interesting part is that it needs no training dataset at all: you record two or three examples of each letter yourself, and it starts recognising them straight away, tracking 21 points on your hand with MediaPipe.
 
We built it for a brief that specifically ruled out using an existing dataset, so most of the work went into making it reliable under real conditions, changing light, hand position, and signs that look almost identical.
 
![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![MediaPipe](https://img.shields.io/badge/MediaPipe-hand%20tracking-orange) ![OpenCV](https://img.shields.io/badge/OpenCV-computer%20vision-green) ![License](https://img.shields.io/badge/License-MIT-lightgrey)
 
**Key concepts:** computer vision, real-time inference, hand landmark detection, few-shot learning, gesture recognition, classification, temporal smoothing
 
![](/images/ngt.gif)
 
