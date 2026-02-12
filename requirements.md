# requirements.md

## 1. Executive Summary

**JanSetu AI** ("People's Bridge") is a multilingual, AI-powered civic assistant designed to bridge the "last-mile delivery gap" in public services for India's 900M+ rural and semi-urban citizens. While governments launch thousands of welfare schemes, uptake remains low due to information asymmetry, complex eligibility criteria, and language barriers. JanSetu AI solves this by providing a conversational, low-bandwidth, and voice-first interface that acts as a personalized "digital social worker."

By leveraging Large Language Models (LLMs) tuned for Indian languages and government domain knowledge, JanSetu AI simplifies discovery, eligibility checking, and application guidance for critical services (health, agriculture, education, finance). The system prioritizes inclusion, operating effectively in low-connectivity environments (2G/3G) and serving users with low digital literacy through intuitive voice interactions.

---

## 2. Problem Statement & Systemic Impact

### The Problem
- **Discovery Deficit**: Citizens are unaware of 70% of schemes they are eligible for.
- **Language Exclusion**: Most official documentation is in bureaucratic English or formal Hindi, alienating dialect speakers.
- **Complexity Barrier**: Eligibility rules are complex (e.g., income thresholds, land holding sizes), leading to high rejection rates.
- **Digital Divide**: Existing portals assume high-bandwidth, desktop access, and text literacy, excluding the most vulnerable.

### Systemic Impact
JanSetu AI aims to:
- Increase scheme saturation rates by **40%** in pilot districts.
- Reduce "agent dependency" and potential exploitation by middlemen.
- Provide policymakers with real-time feedback on scheme accessibility.

---

## 3. Detailed User Personas

### Persona 1: Ramesh (The Farmer)
- **Profile**: 45 years old, smallholder farmer (2 acres), speaks Bhojpuri mixed with Hindi.
- **Tech Literacy**: Low. Uses a basic smartphone for WhatsApp and YouTube. Struggles with typing.
- **Goal**: Wants to know about "PM-KISAN" installments and subsidies for seeds/fertilizers.
- **Key Need**: Voice interface in Bhojpuri; works on patchy network in the fields.

### Persona 2: Priya (The Aspirant)
- **Profile**: 19 years old, college student, first-generation learner.
- **Tech Literacy**: Moderate. Comfortable with apps but intimidated by complex government forms.
- **Goal**: Looking for scholarships for higher education and skill development courses.
- **Key Need**: personalized recommendations based on her marks/caste category; "Explain in Simple Words" for legal terms.

### Persona 3: Anjali (The SHG Leader)
- **Profile**: 34 years old, runs a Self-Help Group (SHG) for village women.
- **Tech Literacy**: Moderate to High (Community mobilizer).
- **Goal**: Wants to find loans for group members and health schemes for pregnant women in her village.
- **Key Need**: Ability to create profiles for others (assisted mode); Community Analytics Dashboard to track village scheme uptake.

---

## 4. Functional Requirements

### 4.1 Onboarding & Profile Management
1.  **REQ-ONB-01**: System must support "Voice-First Onboarding" where users speak their details (Name, Age, Location, Occupation) to create a profile.
2.  **REQ-ONB-02**: System must support typical Indian vernacular login methods (Mobile Number + OTP), with a fallback to WhatsApp-based login.
3.  **REQ-ONB-03**: Users must be able to switch between "Self Mode" and "Assisted Mode" (for SHG leaders helping others).

### 4.2 Scheme Discovery & Search
4.  **REQ-DISC-01**: Users can search via text, voice, or image (e.g., photo of a crop or a medical bill).
5.  **REQ-DISC-02**: System must provide "Proactive Recommendations" based on the user's profile (e.g., "Ramesh, fertilizer subsidies are open this week").
6.  **REQ-DISC-03**: Search results must include a **Compatibility Score** (0-100%) indicating how well the user fits the criteria.

### 4.3 Eligibility Check
7.  **REQ-ELIG-01**: System acts as an interactive interviewer to verify eligibility (e.g., asking "Do you own less than 2 hectares of land?" for PM-KISAN).
8.  **REQ-ELIG-02**: **Innovation**: "Explain in Simple Words" capability to rewrite complex eligibility logic (e.g., "BPL Card holder") into colloquial terms.

### 4.4 Application Guidance
9.  **REQ-APP-01**: Step-by-step "conversational form filling" where the AI asks one question at a time and validates the answer.
10. **REQ-APP-02**: **Innovation**: Document Validation AI allows users to upload a photo of a document (Aadhaar, Income Cert) and get instant feedback on its validity/readability before submission.

### 4.5 Offline & SMS Capabilities
11. **REQ-OFF-01**: **Innovation**: Users can send an SMS with keywords (e.g., "SCHEME CROP") and receive a text summary of top 3 relevant schemes.
12. **REQ-OFF-02**: The app must cache recently viewed schemes and allow full "read-only" access without internet.

---

## 5. Non-Functional Requirements

- **NFR-PERF-01**: The application must load the "Home" interaction state in < 3 seconds on a 2G (64 Kbps) network.
- **NFR-REL-01**: 99.9% Uptime during government working hours (8 AM - 8 PM).
- **NFR-SEC-01**: All Personally Identifiable Information (PII) must be encrypted at rest (AES-256) and in transit (TLS 1.3).
- **NFR-ACC-01**: UI must comply with WCAG 2.1 AA standards (High Contrast, Screen Reader support).
- **NFR-ACC-02**: **Innovation**: "Elder/Parent Mode" with 200% larger text, simplified icons, and high-contrast yellow/black theme.

---

## 6. AI-Specific Requirements

- **AI-RAG-01**: Retrieval Augmented Generation (RAG) system must cite official government URLs for every claim or eligibility rule presented (Hallucination Control).
- **AI-LANG-01**: Support for 12 major Indian languages + 20 distinct dialects (e.g., Maithili, Bhojpuri, Badaga).
- **AI-CONF-01**: **Innovation**: Display an "AI Confidence Score" visible to the user for generated answers (e.g., "High Confidence: This info is from the 2024 Gazette").
- **AI-BIAS-01**: Recommendation engine must be audited to ensure equal visibility for schemes across all genders, castes, and religions.

---

## 7. Data Requirements

- **Sources**: Automated scraping of State and Central Government portals (.gov.in), Official Gazettes, and trusted NGO databases.
- **Update Frequency**: Schemes data refreshed every 24 hours; Application deadlines refreshed hourly.
- **Validation**: "Human-in-the-Loop" dashboard where NGO partners verify new scheme entries before they go live.

---

## 8. Accessibility & Inclusion Requirements

- **Voice-Output**: All text content must have a "Read Aloud" button using high-quality neural TTS.
- **Iconography**: Universal, text-free icons for non-literate navigation (e.g., 'Rupee Symbol' for Finance, 'Tractor' for Agriculture).
- **Error Recovery**: No generic error messages. Audio prompts like "I didn't hear that, please speak closer to the phone" instead of "Input Error."

---

## 9. Ethical & Governance Framework

- **Privacy**: Compliance with the Digital Personal Data Protection (DPDP) Act, 2023. Explicit, granular consent for data usage.
- **Transparency**: Clear labeling of all AI-generated content.
- **Accountability**: Automated "Red Teaming" to test for misinformation regarding political or sensitive topics.

---

## 10. Measurable KPIs & Success Metrics

- **Adoption**: 1 Million Monthly Active Users (MAU) within 18 months.
- **Impact**: ₹500 Crore worth of benefits unlocked for citizens.
- **Engagement**: Average session time > 4 minutes (indicating deep engagement vs bounce).
- **Trust**: > 85% Positive Feedback on "Accuracy of Information."

---

## 11. Risk Analysis & Mitigation

| Risk | Impact | Mitigation Strategy |
| :--- | :--- | :--- |
| **Hallucination** (AI invents a fake scheme) | Critical | Strict RAG pipelines; Only answer from indexed documents; Link to source. |
| **Data Privacy Breach** | Critical | No data retention of user PII after session ends (unless "Save Profile" is checked); Local-first storage. |
| **Low Adoption** | High | Partnership with Common Service Centres (CSCs) and SHGs for trusted ground-level onboarding. |
| **Language Interpretation Error** | Medium | User feedback loop ("Did I understand correctly?") + Fallback to Hindi/English. |

---

## 12. Phased Implementation Roadmap

- **Phase 1: MVP (Months 0-4)**
    - Top 50 Central Schemes.
    - 3 Languages (Hindi, English, Tamil).
    - Basic Voice Search & RAG.
    - Web App (PWA) only.

- **Phase 2: Regional Scale (Months 5-12)**
    - Top 500 Schemes (State + Central).
    - 12 Languages.
    - WhatsApp Integration.
    - Document Validation AI.
    - Offline SMS Mode.

- **Phase 3: National Integration (Months 13-24)**
    - Direct API integration with Gov portals for "One-Click Apply."
    - Community Analytics Dashboard for Government partners.
    - Full Voice-Bot on Toll-Free Number.
