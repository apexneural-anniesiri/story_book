# PRD: AI Personalized Story Book Generator

## 1. Product Overview

- **Project Title:** AI Personalized Story Book Generator  
- **Version:** 1.0  
- **Last Updated:** 2026-02-17  
- **Owner:** ApexNeural  

The **AI Personalized Story Book Generator** creates customized children’s storybooks through two customer channels: **Website** and **WhatsApp**. Parents provide child-specific inputs (photo, name, age, interests, favorite color, favorite cartoons, hobbies), and the product generates a complete, print-ready book with controlled structure and selectable story options.

Each generated book must include exactly:

1. **Front Cover** (dynamic)  
2. **Publisher Page** (dynamic series list per child)  
3. **16 Story Pages** (dynamic)  
4. **Coloring Page** (dynamic black-and-white adaptation)  
5. **Collection Page** (constant)  
6. **Back Page** (semi-dynamic: fixed address + fixed QR code; remaining elements like back-cover photo and moral line are personalized per child/book)

### Authentication & Payments (Required)
- **Authentication:** Users authenticate via the existing ApexNeural SaaS authentication system (login/signup handled by the SaaS).  
- **Payments:** Payments are required to unlock **download** and/or **printed book** fulfillment.
  - Supported gateways: **PayPal** and **Razorpay**.
  - The system must generate a **watermarked preview PDF** first.
  - **Download access** is granted only after successful payment confirmation.

### Pricing Options (Product Rules)
The product must support these purchase options:
1. **Digital PDF Download (Paid)**  
   - User can preview watermarked PDF before payment.  
   - After payment, user can download the **non-watermarked final PDF**.
2. **Printed Book + Shipping (Paid)**  
   - User pays a higher amount for **PDF + printing + shipping**.  
   - User provides shipping address during this purchase flow.  
   - After payment confirmation, order is created for print fulfillment and the user can also download the final PDF.

The product must preserve child-level story history to support future purchases and repeat usage.

The product must support returning members by showing their prior child profiles, **past chat/session history (Website + WhatsApp)**, **books previously purchased/created**, **how many books they have created**, previously generated series, and previous plot selections, so they can create new books either within an existing series (new storyline) or in a different series without re-entering the same information.

**Selection Policy (No Timeouts):**
- There are **no time-window based selections**.
- For series and plot selection, the system must explicitly ask the parent:
  - **Manual Selection** (parent chooses), OR
  - **AI-Assisted Selection** (“Auto Select” → AI chooses best option)
- If the parent does not respond, the flow **waits indefinitely** and does not proceed.

---

## 2. Problem Statement

Personalized children’s books are currently produced manually, which creates:

- Long turnaround time  
- Inconsistent creative quality  
- Poor scalability with order volume  
- Limited reuse of prior child-specific story concepts  

Parents need a fast, consistent, and guided experience to generate personalized books, while the business needs repeatable quality and reusable child story profiles across multiple purchases.

---

## 3. Goals & Objectives

### Business Goals (SMART)

1. **Reduce creation turnaround**  
   Reduce average time from completed parent input to final print-ready **preview PDF** to **≤ 15 minutes for 90% of orders** within **90 days** of launch.

2. **Increase completion rate**  
   Achieve a full-flow completion rate (input to preview PDF) of **≥ 80%** across both Website and WhatsApp channels within **60 days** of launch.

3. **Improve repeat purchase conversion**  
   Achieve **≥ 25% repeat purchases** among customers with at least one prior completed book within **6 months** of launch.

4. **Minimize manual intervention**  
   Keep manual review/edit intervention to **≤ 5%** of generated orders within **90 days** of launch.

5. **Regeneration satisfaction**  
   Ensure **≥ 85%** of users who trigger at least one regeneration complete a final plot selection within **2 regeneration rounds**.

6. **Increase paid conversion from preview**  
   Achieve **≥ 30%** conversion from preview (watermarked PDF generated) to paid purchase (PDF download or print+ship) within **90 days** of launch.

### User Goals

- Provide child details once and receive meaningful story options quickly.  
- Choose preferred series and plot with clear choices and optional AI help when desired.  
- Preview a watermarked book before paying.  
- Pay and immediately download the final PDF (no watermark).  
- Optionally order a printed book with shipping.  
- Reuse prior generated series for faster repeat book creation.  
- View full history of created/purchased books (including count) for each child profile and overall account.

---

## 4. Success Metrics (Quantifiable)

- **Flow Completion Rate (Preview)** = % of started sessions resulting in preview watermarked PDF.  
- **Paid Conversion Rate** = % of preview sessions resulting in successful payment.  
- **PDF Download Unlock Success Rate** = % of paid users who can download final PDF successfully.  
- **Print Order Creation Success Rate** = % of print purchases resulting in a valid print order record.  
- **Median Time to First Series Set** from input submission.  
- **Median Time to Final Plot Selection** from series generation.  
- **AI-Assisted Selection Rate** for series and plot (tracked separately).  
- **Manual Selection Rate** for series and plot (tracked separately).  
- **Regeneration Frequency** = average regeneration attempts per order.  
- **Duplicate Content Violation Rate** = % of regenerated outputs flagged as repeating prior attempts.  
- **Age Appropriateness Compliance Rate** = % of books passing age-fit QA checks.  
- **Face Consistency Pass Rate** across all 16 story pages.  
- **Publisher Page Accuracy Rate** = % with correct primary + complete stored series list.  
- **Constant Page Integrity Rate** = % of books with unmodified collection page and fixed back-page elements (address + QR).  
- **Returning User Reuse Rate** = % of returning members who start a new book using an existing child profile and stored series.  
- **History Visibility Success Rate** = % of returning sessions where past series, past plots, past books, and book counts are visible and correctly linked to the member + child profile.

---

## 5. Target Users & Personas

### Persona 1: Time-Constrained Parent (Primary Buyer)

**Profile:** Parent aged 28–42, buys gifts online, limited time, uses mobile-first workflows.

**Needs:**
- Fast completion in one session  
- Simple guided choices without creative writing effort  
- Confidence the story feels personal and age-appropriate  
- Preview before paying  

**Pain Points:**
- Manual back-and-forth and unclear next steps  
- Delayed delivery from custom products  
- Paying without confidence in output quality  

**Success Criteria:**
- Completes preview in under 20 minutes  
- Understands each selection step clearly  
- Can pay and download instantly  
- Can optionally order print+ship easily  

---

### Persona 2: Repeat Gifting Parent/Guardian (Loyal User)

**Profile:** Parent/relative buying multiple books for same child across occasions.

**Needs:**
- Reuse previously generated series concepts  
- Generate fresh plots without starting from scratch  
- Maintain continuity with child’s personalized story universe  
- See full purchase/creation history and quickly reorder or create again  
- Choose PDF-only or print+ship per occasion  

**Pain Points:**
- Re-entering same data each time  
- Repetitive or duplicated story options  
- Not knowing which books were already created/purchased  
- Friction in repeat payments/orders  

**Success Criteria:**
- Accesses stored child profile and prior series instantly  
- Sees total books created/purchased and per-child counts  
- Completes repeat order with fewer decisions  
- Gets non-repetitive story outcomes  
- Smoothly upgrades from PDF-only to print+ship when desired  

---

## 6. Definitions

- **Primary Series:** The first generated series in a set; shown as recommended (not auto-selected).  
- **Manual Selection:** Parent explicitly selects an option.  
- **AI-Assisted Selection:** Parent explicitly chooses “Auto Select”, and the system selects the best option.  
- **Regeneration Attempt (Version):** A new generation round stored with timestamp and outputs.  
- **Constant Pages:** Collection Page is fully constant; Back Page has fixed elements (address + QR) but other elements are personalized.  
- **Preview PDF:** Watermarked PDF shown to user before payment; not the final downloadable asset.  
- **Final PDF:** Non-watermarked PDF available only after successful payment for PDF or print+ship purchase.  

---

## 7. Features & Requirements

## P0 (Must-Have)

### Feature P0-0: Authentication via Existing SaaS

**Description:** Require user login/signup using ApexNeural SaaS authentication.  
**User Story:** As a user, I want to log in so my children, books, and purchases are saved under my account.

**Acceptance Criteria:**
- [ ] Website requires authentication before accessing saved history and downloads.  
- [ ] WhatsApp identity must map to a SaaS account (existing or newly created/linked).  
- [ ] After authentication, the system can retrieve account history (children, books, purchases).  
- [ ] User can log out (website).  

**Success Metric:** ≥ 99% successful auth sessions without breaking core flow.

---

### Feature P0-1: Child Input Capture (Website + WhatsApp)

**Description:** Collect required parent-provided child data before generation starts.  
**User Story:** As a parent, I want to submit my child’s details once so that the story options are personalized correctly.

**Acceptance Criteria:**
- [ ] System must require all mandatory inputs: child photo, child name, age, interests, favorite color, favorite cartoons, hobbies.  
- [ ] System must block progression until all mandatory inputs are provided.  
- [ ] System must confirm successful input capture before initiating series generation.  
- [ ] Input values displayed back to user must match submitted values exactly.  

**Success Metric:** ≥ 98% of completed sessions begin generation without missing required fields.

---

### Feature P0-2: Generate 5 Original Story Series Concepts

**Description:** Generate exactly 5 distinct, age-appropriate series concepts using only child interests as the theme driver.  
**User Story:** As a parent, I want multiple series options so that I can choose the best creative direction for my child.

**Acceptance Criteria:**
- [ ] Exactly 5 series concepts are generated per generation attempt.  
- [ ] Each series must include a series name and description.  
- [ ] Series must be distinct from one another within the same attempt.  
- [ ] Series must be age-appropriate for provided child age.  
- [ ] The first generated series is tagged as **Primary Series (recommended)**.  

**Success Metric:** ≥ 95% of attempts produce 5 valid, distinct, age-appropriate series.

---

### Feature P0-3: Series Storage & Primary Tagging

**Description:** Persist all generated series under the child profile for current and future reuse.  
**User Story:** As a repeat buyer, I want past series retained so I can reuse and build on prior story concepts.

**Acceptance Criteria:**
- [ ] Each generated series record stores: series name, series description, date created, selection status, and selection method (Manual vs AI-Assisted).  
- [ ] Primary Series indicator is stored and visible for each generation attempt.  
- [ ] All generated series are retrievable under the specific child profile.  
- [ ] Series records from separate regeneration attempts remain separate and traceable by attempt/version.  

**Success Metric:** 100% of generated series are retrievable with correct metadata.

---

### Feature P0-4: Series Selection (Manual or AI-Assisted, No Timeouts)

**Description:** Parent either selects a series manually or explicitly requests AI to select the best series.  
**User Story:** As a parent, I want control over series selection, and I want AI to select only if I request it.

**Acceptance Criteria:**
- [ ] Parent can choose **Manual Selection** or **AI-Assisted Selection (Auto Select)**.  
- [ ] Parent can select exactly one of the 5 generated series in Manual Selection.  
- [ ] If parent chooses AI-Assisted Selection, the system selects exactly one series and explains why it was selected (short rationale).  
- [ ] The system must not select any series unless the parent explicitly chooses Manual or AI-Assisted Selection.  
- [ ] The system waits indefinitely (no timeout) until the parent makes a selection decision.  
- [ ] Final selected series status and selection method are stored in records.  

**Success Metric:** 100% of flows end with exactly one selected series with selection method recorded.

---

### Feature P0-5: Generate 5 Plots from Selected Series

**Description:** Generate 5 plot options for the selected series.  
**User Story:** As a parent, I want multiple plot choices so that I can pick the emotional story arc I prefer.

**Acceptance Criteria:**
- [ ] Exactly 5 plots are generated per selected series attempt.  
- [ ] Each plot includes: storyline summary, beginning, middle, end, emotional arc, age-appropriate tone.  
- [ ] All plots are stored under the selected series record.  
- [ ] Plot options are distinct from one another.  

**Success Metric:** ≥ 95% of plot-generation attempts return 5 complete, valid plot records.

---

### Feature P0-6: Plot Selection (Manual or AI-Assisted, No Timeouts)

**Description:** Parent either selects a plot manually or explicitly requests AI to select the best plot.  
**User Story:** As a parent, I want to choose the final story direction, and I want AI to select only if I request it.

**Acceptance Criteria:**
- [ ] Parent can choose **Manual Selection** or **AI-Assisted Selection (Auto Select)**.  
- [ ] Parent can select exactly one of 5 plots in Manual Selection.  
- [ ] If parent chooses AI-Assisted Selection, the system selects exactly one plot and explains why it was selected (short rationale).  
- [ ] The system must not select any plot unless the parent explicitly chooses Manual or AI-Assisted Selection.  
- [ ] The system waits indefinitely (no timeout) until the parent makes a selection decision.  
- [ ] Selected plot is stored with selected status and selection method.  
- [ ] User receives confirmation of selected plot method (Manual vs AI-Assisted).  

**Success Metric:** 100% of flows end with exactly one selected plot with selection method recorded.

---

### Feature P0-7: Regeneration Logic with Version History

**Description:** Support controlled regeneration of plots and full series+plot sets without repeating prior outputs.  
**User Story:** As a parent, I want new options when I dislike current suggestions so I can still find a story I love.

**Acceptance Criteria:**
- [ ] If user rejects all 5 plots, system regenerates 5 new plots for the same selected series.  
- [ ] If user rejects regenerated plots again, system regenerates 5 new series concepts and associated plot options.  
- [ ] Regenerated outputs must not repeat prior outputs for the same child flow.  
- [ ] Each regeneration attempt is stored as a separate version with timestamp and selection outcomes.  
- [ ] Age appropriateness remains enforced across every regeneration attempt.  

**Success Metric:** Duplicate-content violation rate in regeneration ≤ 2%.

---

### Feature P0-8: Final 16-Page Story Generation

**Description:** Create the full story content using selected plot and required narrative rules.  
**User Story:** As a parent, I want the final story to feel coherent and age-fit page by page.

**Acceptance Criteria:**
- [ ] Story has exactly 16 story pages.  
- [ ] Page 1 clearly introduces the child.  
- [ ] Each story page contains maximum 2–3 lines.  
- [ ] Language is understandable by an 8-year-old.  
- [ ] Story ends with a moral in maximum 2 lines.  
- [ ] Narrative flow is coherent from beginning to end.  

**Success Metric:** ≥ 95% of generated books pass content QA checklist on first pass.

---

### Feature P0-9: Illustration Consistency & Style Compliance

**Description:** Generate illustrations that match required style and preserve child face consistency across pages.  
**User Story:** As a parent, I want my child to look recognizably the same throughout the book.

**Acceptance Criteria:**
- [ ] Illustration style is semi-realistic cartoon with target blend of 70% photorealistic / 30% stylized.  
- [ ] Child facial identity remains consistent across all 16 story pages.  
- [ ] Character proportions remain natural throughout.  
- [ ] Coloring page is produced as black-and-white adaptation.  

**Success Metric:** Face-consistency pass rate ≥ 95% in QA sampling.

---

### Feature P0-10: Publisher Page Dynamic Population

**Description:** Build publisher page using stored series records while keeping fixed layout requirements.  
**User Story:** As a buyer, I want the publisher page to accurately reflect my child’s personalized series catalog.

**Acceptance Criteria:**
- [ ] Publisher page shows Primary Series at the top.  
- [ ] Publisher page lists all series generated for the child profile.  
- [ ] Publisher page includes required copyright text.  
- [ ] Publisher page includes required branding.  
- [ ] Layout remains fixed while series list content is dynamic.  

**Success Metric:** Publisher page accuracy rate ≥ 99%.

---

### Feature P0-11: Constant Collection & Back Page Enforcement

**Description:** Ensure collection page is constant and back page fixed elements remain unchanged.  
**User Story:** As the business owner, I want brand-control elements to stay consistent in every generated book.

**Acceptance Criteria:**
- [ ] Collection page layout and content are identical across all generated books.  
- [ ] Back page always contains fixed ApexNeural address (unchanged).  
- [ ] Back page always contains fixed QR code (unchanged).  
- [ ] Back page may include personalized elements (e.g., back-cover photo and moral line) but must not modify the fixed address + QR.  

**Success Metric:** Fixed-element integrity rate (collection + back QR/address) = 100%.

---

### Feature P0-12: Preview PDF Generation (Watermarked)

**Description:** Generate a watermarked preview PDF first, before payment, for user review.  
**User Story:** As a parent, I want to preview the book before paying so I know it looks good.

**Acceptance Criteria:**
- [ ] System generates a preview PDF that includes all required sections in correct sequence.  
- [ ] Preview PDF is watermarked clearly on every page (except constant rules if any).  
- [ ] Preview PDF is viewable in the initiating channel (Website view or WhatsApp link).  
- [ ] Preview state is stored in the book record.  

**Success Metric:** Preview PDF generation success rate ≥ 99%.

---

### Feature P0-13: Payments & Unlocking (PayPal + Razorpay)

**Description:** Support payment to unlock final PDF download and/or print+ship ordering.  
**User Story:** As a parent, I want to pay securely and then immediately download or order a printed book.

**Acceptance Criteria:**
- [ ] User can choose purchase option: **PDF Download** or **Print+Ship (includes PDF)**.  
- [ ] Payments can be processed via PayPal or Razorpay.  
- [ ] System confirms payment success before granting access.  
- [ ] If payment fails/cancels, user remains on preview with retry options.  
- [ ] After successful payment for PDF Download, user can download the final non-watermarked PDF.  
- [ ] After successful payment for Print+Ship, user can download the final PDF and a print order is created.  

**Success Metric:** Payment success-to-unlock success rate ≥ 99%.

---

### Feature P0-14: Final PDF Delivery (Non-Watermarked)

**Description:** Provide final non-watermarked PDF only after payment confirmation.  
**User Story:** As a parent, I want the final PDF after payment so I can print or save it.

**Acceptance Criteria:**
- [ ] Final PDF is not downloadable before payment success.  
- [ ] After payment, final PDF is available via website download (and link confirmation via WhatsApp if applicable).  
- [ ] Download access persists in user history for re-download.  
- [ ] System shows “Paid/Unlocked” status on the book record.  

**Success Metric:** Final PDF delivery success rate ≥ 99%.

---

### Feature P0-15: Print + Shipping Order (Paid Upgrade)

**Description:** Offer print+ship option with address capture and order creation after payment.  
**User Story:** As a parent, I want a printed book delivered to my address.

**Acceptance Criteria:**
- [ ] Print+Ship option requires shipping address entry before payment completion.  
- [ ] After payment confirmation, an order is created with address and book reference.  
- [ ] User can view print order status in history (at minimum: “Order Placed”).  
- [ ] Print purchase always includes access to final PDF.  

**Success Metric:** Print order creation success rate ≥ 99%.

---

### Feature P0-16: Returning Member Access to Past History (Website + WhatsApp)

**Description:** Provide returning members access to stored child profiles, chat/session history summaries, generated series, selected plots, and completed/purchased books, including counts.

**User Story:** As a returning parent, I want to see my previous books and series (and how many I created) so that I can create a new book quickly without starting from scratch.

**Acceptance Criteria:**
- [ ] Returning member can view a list of previously created child profiles.  
- [ ] For each child profile, the system shows previously generated series (including Primary tag and timestamps).  
- [ ] For each child profile, the system shows completed books with: book title, series used, creation date, and purchase status (Preview / Paid PDF / Paid Print).  
- [ ] System shows total number of books created/purchased for the member account.  
- [ ] System shows per-child book count for each child profile.  
- [ ] System shows past conversation/session history summaries (e.g., last generation date, selected series, selected plot).  
- [ ] Users can re-download any previously paid PDF.  

**Success Metric:** History visibility success rate ≥ 99%.

---

### Feature P0-17: Create New Book from Existing Series OR Different Series

**Description:** Allow returning members to create a new book using either:
- An existing stored series (generate new plots/storylines), or
- A new series generation flow (5 new series concepts)

**User Story:** As a returning parent, I want to create another book for my child in the same series or a different series so that the child gets fresh stories while keeping continuity.

**Acceptance Criteria:**
- [ ] User can start a new book from an existing child profile without re-entering unchanged inputs.  
- [ ] User can choose one of the existing stored series and request new plot options for that series.  
- [ ] User can choose to generate a new set of 5 series concepts instead of using stored series.  
- [ ] Every newly generated series and every newly completed book must be stored under the child profile and visible in history.  
- [ ] The system must prevent accidental duplication by warning if the user attempts to select a plot that is already used in a prior completed book for the same child.  

**Success Metric:** ≥ 25% of returning users complete a new book using stored series or profiles within 6 months of launch.

---

## P1 (Should-Have)

### Feature P1-1: Repeat Purchase from Stored Child Profile

**Description:** Allow returning users to generate new books using prior child profile and stored series records.  
**User Story:** As a repeat buyer, I want faster checkout by reusing existing child context.

**Acceptance Criteria:**
- [ ] User can select an existing child profile.  
- [ ] Stored series history is visible before new generation starts.  
- [ ] User can proceed with new generation without resubmitting unchanged child details.  
- [ ] New outputs are stored as a separate purchase flow.  

**Success Metric:** Repeat-flow completion time is ≥ 30% faster than first-time flow median.

---

### Feature P1-2: Gentle Reminder After Inactivity (No Auto-Selection)

**Description:** Notify user after prolonged inactivity that a selection is pending, without applying any automatic selection.  
**User Story:** As a parent, I want a reminder so I don’t forget to complete my selection.

**Acceptance Criteria:**
- [ ] Reminder can be enabled/disabled by the user or business rules.  
- [ ] Reminder message states that the flow is waiting for the user’s selection.  
- [ ] Reminder must not trigger any automatic selection.  
- [ ] Reminder delivery is logged per flow.  

**Success Metric:** ≥ 40% of reminder recipients return to complete selection within a defined observation window.

---

### Feature P1-3: Progress Visibility Across Flow

**Description:** Show clear status stages from input to preview PDF, payment, and final delivery.  
**User Story:** As a parent, I want to know where I am in the process and what is pending.

**Acceptance Criteria:**
- [ ] User sees current stage label at each step.  
- [ ] User sees whether action is required from them.  
- [ ] User sees completion confirmation for each finished stage.  
- [ ] Payment stage is clearly shown as required to unlock download/print.  
- [ ] Final stage confirms PDF readiness and/or print order placement.  

**Success Metric:** User-reported clarity score ≥ 4.2/5 in post-order survey.

---

## P2 (Nice-to-Have)

### Feature P2-1: Preferred Tone Presets for Story Mood

**Description:** Offer optional tone presets (e.g., adventurous, gentle, funny) during plot review.  
**User Story:** As a parent, I want to bias plot mood toward my child’s personality.

**Acceptance Criteria:**
- [ ] Tone preset selection is optional.  
- [ ] Selected tone visibly influences regenerated plot style.  
- [ ] Core age-appropriateness and structure requirements remain unchanged.  

**Success Metric:** ≥ 20% adoption among eligible users with no drop in completion rate.

---

### Feature P2-2: Occasion Tagging for Repeat Books

**Description:** Tag book generation by occasion (birthday, festival, milestone) for user organization.  
**User Story:** As a repeat buyer, I want to track why each book was created.

**Acceptance Criteria:**
- [ ] Occasion selection is optional.  
- [ ] Occasion appears in order history label.  
- [ ] Occasion does not alter constant-page constraints.  

**Success Metric:** ≥ 30% of repeat orders include an occasion tag.

---

## 8. Explicitly Out of Scope

> This section is binding for MVP and V1.0 unless explicitly updated in this PRD.

- Multi-child storybooks in one single book instance.  
- Parent-authored custom text editing of generated page content.  
- Voice narration or audiobook production.  
- Real-time co-authoring between multiple family members.  
- Marketplace for third-party story templates.  
- Loyalty programs, coupons, or referral systems.  
- Social sharing feed/community features.  
- Translation/localization to multiple languages at launch.  
- Merchandise generation (posters, toys, stickers) from book content.  
- Offline/manual payment methods outside PayPal/Razorpay.  

---

## 9. User Scenarios

### Scenario A: Website Full Flow (Manual Selection → Preview → Pay PDF → Download)

**Context:** First-time parent uses website to create a book and buys PDF.

**Steps:**
1. Parent opens website and logs in via SaaS authentication.  
2. Parent starts new book.  
3. Parent submits mandatory child inputs.  
4. System generates 5 series; first is marked Primary Series (recommended).  
5. System asks parent: Manual Selection or AI-Assisted Selection.  
6. Parent manually selects one series.  
7. System generates 5 plots for selected series.  
8. System asks parent: Manual Selection or AI-Assisted Selection.  
9. Parent manually selects one plot.  
10. System generates preview book and returns **watermarked preview PDF**.  
11. Parent chooses **PDF Download** and pays via PayPal/Razorpay.  
12. After payment success, system unlocks and provides **final non-watermarked PDF** download.

**Expected Outcome:**
- Parent previews, pays, and downloads final PDF.  
- Book history stores: preview + paid status + download availability.

**Edge Cases & Error Handling:**
- Payment failed/cancelled: user remains at preview with retry option.  
- Missing input fields: progression blocked with clear prompts.  
- Parent inactivity during selection: flow remains pending; optional reminder may be sent (no auto-selection).  

---

### Scenario B: Website Full Flow (AI-Assisted Selection → Preview → Pay Print+Ship)

**Context:** Parent wants AI to choose and orders printed book.

**Steps:**
1. Parent logs in.  
2. Parent submits child inputs.  
3. System generates 5 series → parent chooses AI-Assisted Selection → system selects and explains.  
4. System generates 5 plots → parent chooses AI-Assisted Selection → system selects and explains.  
5. System generates **watermarked preview PDF**.  
6. Parent chooses **Print+Ship**.  
7. Parent enters shipping address.  
8. Parent pays higher amount via PayPal/Razorpay.  
9. After payment success, system:
   - unlocks final PDF download, and  
   - creates a print+shipping order record.

**Expected Outcome:**
- Parent receives final PDF access and print order confirmation.

**Edge Cases & Error Handling:**
- Address incomplete: block payment until fixed.  
- Payment success but order creation fails: show “Support required” state and log incident.

---

### Scenario C: WhatsApp Flow (Manual Selection → Preview → Pay → Unlock)

**Context:** Parent creates book via WhatsApp and purchases PDF.

**Steps:**
1. Parent initiates WhatsApp conversation.  
2. System links/creates account (SaaS authentication mapping).  
3. System collects child inputs.  
4. System sends 5 series choices and asks Manual vs AI-Assisted.  
5. Parent selects one.  
6. System sends 5 plot choices and asks Manual vs AI-Assisted.  
7. Parent selects one.  
8. System generates **watermarked preview PDF** and shares preview link.  
9. Parent chooses PDF or Print+Ship, completes payment via PayPal/Razorpay link.  
10. On payment success, system shares final download link (and confirms print order if chosen).

**Expected Outcome:**
- WhatsApp user can preview, pay, and receive final asset access.

---

## 10. Dependencies & Constraints

- Users must authenticate via existing SaaS authentication.  
- Parent must provide a usable child photo and complete mandatory fields.  
- Preview PDF must be generated before payment.  
- Final non-watermarked PDF must be locked until payment success.  
- Print+ship requires shipping address capture and a higher payment.  
- Payments must support PayPal and Razorpay.  
- Website and WhatsApp must both support full generation lifecycle.  
- The system must not proceed past series/plot selection without an explicit parent decision (Manual vs AI-Assisted).  
- All generation attempts and selections must be persistently available for repeat usage.  
- Returning members must be able to access their stored child profiles, series history, chat/session summaries, and prior purchases across both channels.

---

## 11. Timeline & Milestones

### Milestone 1: Requirements Finalization (Week 1)
- Final sign-off on flow rules, page constraints, watermark preview + payment unlock rules.  
- Acceptance checklists approved for each P0 feature.

### Milestone 2: End-to-End P0 Readiness (Weeks 2–5)
- Website and WhatsApp both support full P0 flow.  
- Series/plot generation, manual selection, AI-assisted selection, regeneration, preview PDF, payment unlock, final PDF delivery, print order creation.

### Milestone 3: QA & UAT Validation (Weeks 6–7)
- Validate acceptance criteria with scenario-based testing.  
- Confirm watermark enforcement and paid unlock enforcement.

### Milestone 4: Production Launch (Week 8)
- Go-live for P0 features across both channels.  
- Metrics dashboard active for success metrics.

### Milestone 5: P1 Enhancements (Weeks 9–11)
- Repeat purchase acceleration, reminders, progress visibility improvements.

### Milestone 6: P2 Experimentation (Weeks 12+)
- Optional tone presets and occasion tagging tested and iterated.

---

## 12. Risks & Assumptions

### Risks
- Preview watermark could be bypassed if download protection is weak.  
- Payment confirmation delays may block timely unlock.  
- Inconsistent child-face likeness could reduce trust in personalization quality.  
- Print order handoff failures could cause customer dissatisfaction.  
- User drop-off may occur between preview and payment.

### Assumptions
- SaaS authentication system is stable and supports website + WhatsApp identity mapping.  
- PayPal and Razorpay support required regions/currencies for your customers.  
- Print and shipping operations exist as a downstream process (even if not detailed here).  
- Parents are willing to preview then pay for downloads/prints.  

---

## 13. Non-Functional Requirements

- **Reliability:** ≥ 99% successful completion for preview PDF generation once plot is selected.  
- **Unlock Integrity:** 100% enforcement that final PDF download is inaccessible before payment.  
- **Payment Integrity:** ≥ 99% successful payment-to-unlock linkage (no paid users blocked).  
- **Print Order Integrity:** ≥ 99% successful print order creation after print+ship payment.  
- **Data Persistence:** 100% retention of series/plot/version history per child profile, and retention of purchase history including counts.  
- **Usability:** User-facing instructions must be understandable at grade-6 reading level.  
- **Traceability:** Every selection and payment event must be auditable by timestamp and method.  
- **Performance:** Initial 5-series generation should complete within 60 seconds for 90% of requests.  
- **Scalability:** Must support peak seasonal demand without changing product behavior.  
- **Content Safety:** 100% of published outputs must pass age-appropriateness gating.  
- **Channel Parity:** Core flow outcomes must be functionally equivalent on Website and WhatsApp.

---

## 14. References

- Existing manually created personalized book PDFs (structure and pacing reference).  
- Brand-approved fixed Collection Page master.  
- Brand-approved fixed Back Page fixed elements (address + QR code).  
- Internal age-appropriateness and emotional-tone content guidelines.  
- Publisher page branding and copyright standard text guidelines.
