---
title: About Me
layout: splash
permalink: /about/
collection: about
entries_layout: grid
classes: wide
header:
  image: /images/about_1.gif
  teaser: /images/about_1.gif
# gallery:
#   - url: /images/portfolio/kwdc.png
#     image_path: /images/portfolio/kwdc.png
#     alt: "placeholder image 3"
#   - url: /images/portfolio/deep.jpg
#     image_path: /images/portfolio/deep.jpg
#     alt: "placeholder image 1"
#   - url: /images/portfolio/start.jpg
#     image_path: /images/portfolio/start.jpg
#     alt: "placeholder image 3"
#   - url: /images/portfolio/hackjudge.jpg
#     image_path: /images/portfolio/hackjudge.jpg
#     alt: "placeholder image 2"
#   - url: /images/portfolio/hack.JPG
#     image_path: /images/portfolio/hack.JPG
#     alt: "placeholder image 3"
#   - url: /images/portfolio/idea.JPG
#     image_path: /images/portfolio/idea.JPG
#     alt: "placeholder image 3"
---

Hi, I'm **Dongchan Kim** (Alex). <br>
I'm a **Software Engineer** studying Computer Science at Stony Brook University, with professional experience in real-time systems, backend infrastructure, and XR development. I build things that scale — from syncing 200+ VR headsets in a live concert venue to deploying multi-modal AI servers that respond in under 5 seconds.

---

## Education

+ **The State University of New York, Stony Brook University** — B.S. in Technological Systems Management, Specialized in *Computer Science* (GPA: 3.81 / 4.00) **Aug 2024 – May 2026**
+ **Ghent University Global Campus** — Transferred to Stony Brook University **Mar 2019 – Jun 2024**

---

## Professional Experience

+ **Software Engineer**, **KAI.INC** — SM REALIVE K-Pop VR Theater System **Jun 2025 – Oct 2025**
  + Designed a precision sync algorithm using RTT to align playback across 200+ VR HMDs within ±100ms, stabilizing inter-client lag to 1–2 frames (~33–66ms) in Alpha & Beta testing
  + Refactored a 3,000-line monolithic controller into four independent managers using SRP and Dependency Injection; resolved memory leaks via asynchronous loading and forced GC
+ **Technical Education Lead**, **TeamSparta Inc.** **Jul 2024 – Aug 2025**
  + Orchestrated core CS curriculum for 500+ developers, boosting average coding test pass rates by 25%; selected as "Best Tech Education Leader"
  + Conducted deep code reviews for 1,000+ implementations using Clean Code principles, achieving a 30% average gain in execution speed and memory efficiency

---

## Awards & Honors

+ **MIT Reality Hack 2026**, Cambridge — **Grand Gold Award & Meta Track Winner** (Team of 5)
+ **2024 Google Saessak Hackathon**, Seoul — **Top Honorable Mention Winner** (Team of 5)
+ **Machine Learning Specialization** — Certificate
+ Seoul National University OUTTA AI Deep Learning Camp — **Double Award Winner** (Participant Excellence, Team Excellence)
+ IGC Startup Idea Competition — **Grand 2nd Prize Winner**
+ Ministry of Science and ICT, Youth SW Mentorship Hackathon — Served as **Judge and Mentor Coach**

---

## Technical Skills

+ `Python` / Backend, Automation, Pytest, BeautifulSoup
+ `JavaScript` / Node.js, React.js, WebSocket
+ `C#` / Unity, XR Development
+ `Infrastructure` / AWS (EC2, RDS, S3), Docker, GitHub Actions, CI/CD
+ `Database` / PostgreSQL, MySQL, RESTful API Design
+ `Others` / Bash/Shell Scripting, Linux, Cron, Swagger

---

## Projects

### SmartSight — MIT Reality Hack 2026 Gold Award & Meta Track Winner
`Node.js` `PostgreSQL` `CI/CD` `WebSocket` `GPT-4.1 Vision`

+ Developed a multi-modal AI server integrating GPT-4.1 Vision (HTTP) and OpenAI Realtime Voice API (WebSockets), enabling seamless real-time voice interaction and visual environment analysis with minimal latency
+ Reduced end-to-end latency by 60% (13s → 5s) under extreme network congestion; engineered asynchronous logic to decouple real-time voice from database I/O
+ Transitioned from single-user SQLite to production-grade AWS RDS (PostgreSQL) cluster with optimized database schema and connection pooling for concurrent multi-user transactions
+ Established a fully automated CI/CD pipeline via GitHub Actions and Docker; achieved 100% repeatable production deployments

### SecureSBU — 2025 SBUhack, Team Lead
`Node.js` `Express` `MySQL` `CI/CD` `Jest` `Swagger` `Discord.js`

+ Engineered a modular Layered Architecture using Node.js/Express; implemented advanced pagination and filtering that improved data retrieval speed by 40% for large-scale security datasets
+ Hardened system integrity via SQL injection prevention; integrated Discord.js for real-time alerting, reducing incident response latency by over 80%
+ Established an automated pipeline via GitHub Actions and Jest; spearheaded Swagger automation, reducing team collaboration overhead by 30%

### Web Content Integrity Monitor & Automation Pipeline
`Python` `Pytest` `BeautifulSoup` `Cron`

+ Engineered a custom HTML-to-Markdown engine with 95% extraction accuracy; implemented a robust testing suite using Pytest to validate parsing logic against edge cases, reducing production runtime errors by 40%
+ Established a 100% data integrity mapping system; developed an N-day difference detection pipeline that monitors hundreds of snapshots with 0% false-positives

---

## Leadership

+ **Vice President**, [**Lambda at SUNY Korea**](http://lambda-idea.com/about) — Code2Career Team Leader **Aug 2024 – Jun 2025**
  + Led data-driven outreach and strategic planning for club recruitment, resulting in a 350% YoY increase in new member applications
+ **Junior Developer**, [**XREAL**](https://www.xreal.info/) — XR Developer Group at Seoul National University
+ **Student Council** — Junior Year Representative
+ **2024 AI Education City Program** — Academic Research Mentor for Incheon Jinsan Science High School
+ **Republic of Korea Army** — Completed Military Service as a Medic

---

## Hobbies

<div style="display: flex; gap: 10px; justify-content: center;">
  <img src="/images/about2.jpg" style="width: 48%;">
  <img src="/images/rooney.JPG" style="width: 48%;">
</div>

I've been a dedicated fan of **Manchester United** for over 10 years and also root for the esports team **HLE (Hanwha Life Esports)**!

---

## Contact
+ <dck.alx@gmail.com>
+ [LinkedIn](https://www.linkedin.com/in/teddy-lee/)
+ [GitHub @dongckim](https://github.com/dongckim)
+ [Portfolio](https://dongckim.github.io/)
