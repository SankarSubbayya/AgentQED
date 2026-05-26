# Competition Analysis — Zero to Agent Hackathon

## Team: Alpha Vector

**Project:** Alpha Vector — AI analyst and intelligence platform for public health

**What It Does:**
- Maps environmental data across Southeast Asia for disease outbreak detection
- Public health professionals click on regions to get AI-generated health summaries from Gemini
- An agent that's "intimately integrated into the environment" — it can move around the map, answer questions, and pull data from WHO + environmental sensors
- Uses embeddings from "Alpha Curve" (another foundation model) combined with environmental data

**Tech Stack:**
- Gemini (for text summaries and Q&A)
- Alpha Curve embeddings (environmental/health data)
- Map-based UI with interactive data points
- Multi-modal: text, numbers, vector embeddings, geographic data

**Demo Flow (2 min 15 sec):**
1. Shows analyst platform with map of Southeast Asia
2. Clicks on data points → shows environmental data + Gemini summary
3. Asks agent about Dengue → agent moves to relevant area, shows risk info
4. Asks about Malaria → agent navigates map and pulls WHO data
5. Emphasizes multi-modal capability

**Strengths:**
- Real-world impact (public health, disease outbreaks)
- Multi-modal data integration (text + numbers + embeddings + geography)
- Visually impressive map-based demo
- Agent acts autonomously (moves around map, finds relevant data)

**Weaknesses:**
- Unclear how much was built during the hackathon vs pre-existing
- "Alpha Curve" embeddings seem pre-built — how much is new?
- No clear Vercel stack usage mentioned (AI SDK? Sandbox? Workflows?)

**How AgentQED Compares:**
- We have stronger Vercel integration (AI SDK agent loop, Sandbox, deployment)
- We have formal verification (compiler-verified, not just LLM output)
- They have stronger impact story (public health > math proofs)
- We have more input modalities (text + voice + image + PDF vs their text + map clicks)
- Our self-correction loop is more technically novel
