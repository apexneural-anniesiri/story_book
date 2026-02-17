# Application Flow Documentation

## 1. Entry Points

### Website Entry Points
- **Primary Entry Points**
  - Direct URL to Home page.
  - Direct URL to Login (Email + Password) page (ApexNeural SaaS authentication) for existing users.
  - Direct URL to Sign Up (Email Registration) for new users.
  - Direct URL to Dashboard for authenticated returning users (if unauthenticated, redirect to Login (Email + Password)).
- **Secondary Entry Points**
  - Public marketing pages: About, Stories Preview Gallery, Contact/Support, Careers, Terms & Conditions, Privacy Policy, Refund/Shipping Policy.
  - Forgot Password link under Login for password reset via email.
  - Campaign links from email/WhatsApp that deep-link to authenticated pages (Preview Viewer, Checkout/Payment, My Books/Library, Book Detail).
  - SEO landing pages that route to Home and then Login/Sign Up flow.

### OAuth/Social Login
- Not supported in V1.

### WhatsApp Entry Points
- **Primary Entry Points**
  - User sends a start keyword (e.g., “Hi”, “Start”, “Book”) to official WhatsApp number.
  - User taps a “Start on WhatsApp” CTA from website or campaign.
- **Secondary Entry Points**
  - User replies to a previously active conversation.
  - User opens a WhatsApp template message link from campaigns.
- **Conversation Trigger**
  - On first recognized initiation message for a user session, AI sends the mandatory first formal onboarding message.

### First WhatsApp AI Message Template
> **Hello and welcome to ApexNeural Personalized Story Books.**  
> We are delighted to help you create a custom story book for your child.  
> To begin, please keep the following ready: your child’s name, age, interests, favorite color, favorite cartoons, hobbies, and one clear child photo.  
> **Photo consent note:** By continuing, you confirm that you are authorized to share this child photo for personalized book creation.  
> If you are new, you will create an account using your email and password in the next step.  
> Please tap **Start** to continue.

- **Message Trigger:** Sent immediately after first valid conversation initiation or keyword detection.
- **CTA:** `Start`

## 2. Core User Flows

### Primary User Goals
- Create a personalized story book for a child (name, age, interests, photo).
- Select one series and one plot from AI-generated options (manual or AI-assisted).
- View watermarked preview before payment.
- Purchase PDF and/or Print+Ship; receive final PDF and (if Print+Ship) print order.
- Manage children profiles, books, and account (login, signup, password, settings).

### Flow 1: Website Authentication (Email + Password / Email Signup) → Dashboard
- **Goal:** Enforce account-based access before any personalized data or purchase actions (P0-0).
- **Entry Point:** Website Home, Login deep link, Sign Up link, or protected route redirect.
- **Frequency:** Every new session requiring protected data.

#### Happy Path
1. `WEB_PUBLIC_HOME`: user clicks Login (Email + Password) or Sign Up (Email Registration).
   - **Elements:** Home page, Login CTA, Sign Up CTA.
   - **User Action:** Click Login or Sign Up.
   - **System Response/Trigger:** Navigate to login or signup screen.
2. **Existing user:** `WEB_AUTH_LOGIN`: user logs in with email + password successfully.
   - **Elements:** Email + password form, Forgot Password link.
   - **User Action:** Enter credentials and submit.
   - **System Response/Trigger:** Validate; on success create session and redirect to Dashboard.
3. **New user:** `WEB_AUTH_SIGNUP`: user signs up using email + password registration.
   - **Elements:** Email + password signup form.
   - **User Action:** Enter email and password, submit.
   - **System Response/Trigger:** Register account; on success create session and redirect to Dashboard.
4. **Forgot Password:** user may use Forgot Password link from Login; system sends reset link to email; user completes reset on `WEB_AUTH_RESET_PASSWORD`.
   - **Elements:** Forgot Password link, email input, reset form (from email link).
   - **User Action:** Request reset; receive email; open link and set new password.
   - **System Response/Trigger:** Send reset link; accept token and new password; confirm and redirect to Login.
5. **Change Password:** available inside Account Settings (`WEB_ACCOUNT_SETTINGS`) for authenticated users.
   - **Elements:** Change Password option in Account Settings.
   - **User Action:** Enter current and new password, submit.
   - **System Response/Trigger:** Validate and update password; show success or error.
6. `WEB_DASHBOARD`: system loads account summary (My Children count, My Books count, recent history, Start New Book).
   - **Elements:** Summary counts, recent history, Start New Book CTA.
   - **User Action:** (None required; page loads.)
   - **System Response/Trigger:** Load account data and render dashboard.

#### Error States
- Invalid credentials (email/password).
- Email already registered (during signup).
- Password validation failure.
- Reset link expired.
- Account locked or unavailable.
- Authentication service unavailable.

#### Edge Cases
- Protected deep link opened while unauthenticated → redirect to Login (Email + Password), then return.
- Already-authenticated user opens login route → redirect to Dashboard.

#### Exit Points
- Continue to My Children, My Books, Create New Book, Settings, or Support.

### Flow 2: WhatsApp onboarding/account mapping → Start new book
- **Goal:** Map WhatsApp identity to email-based ApexNeural SaaS account before story creation (P0-0). Mapping must be completed before any child or book actions.
- **Entry Point:** WhatsApp first message/template.
- **Frequency:** First interaction per user; re-validated for returning sessions.

#### Happy Path
1. `WA_INIT`: system sends formal onboarding message; user taps Start.
2. `WA_ACCOUNT_MAPPING`: user completes account mapping:
   - **If user has account:** login with email + password.
   - **If new user:** sign up using email + password.
3. Mapping must be completed before any child/book actions.
4. `WA_DASHBOARD_SUMMARY`: system shows My Children, My Books, and Start New Book options.

#### Error States
- Mapping failed.
- Duplicate/conflicting identity.
- Account service unavailable.

#### Edge Cases
- Returning mapped user resumes without remapping.
- Mapped user with zero children/books.

#### Exit Points
- Continue to child select/create.

### Flow 3: Create/select child profile (website + WhatsApp parity)
- **Goal:** Use existing child profile or create a new profile for book flow (P0-16, P0-17).
- **Entry Point:** Dashboard (web or WhatsApp summary).
- **Frequency:** Every new book creation.

#### Happy Path
1. `CHILD_PROFILE_PICKER`: user selects existing child or Create New Child.
2. `CHILD_PROFILE_CREATE_EDIT`: user enters/saves profile details.
3. `BOOK_CREATION_START`: child context loads for generation flow.

#### Error States
- Missing required profile fields.
- Save failed.

#### Edge Cases
- Multiple children; user switches target child.
- Returning user starts a new book from existing child.

#### Exit Points
- Proceed to mandatory input capture.

### Flow 4: Capture child inputs (name/age first on WhatsApp) + photo validation
- **Goal:** Collect all mandatory child inputs before series generation (P0-1).
- **Entry Point:** Book creation start.
- **Frequency:** Every new book unless unchanged stored values are reused.

#### Happy Path
1. `INPUT_CAPTURE_START`: user provides inputs.
   - **Elements:** Input fields (name, age, interests, favorite color, favorite cartoons, hobbies), photo upload. WhatsApp order is strict: name → age → details → photo → optional more details.
   - **User Action:** Enter or upload each mandatory and optional value.
   - **System Response/Trigger:** Accept and store inputs; prompt next field (WhatsApp) or show summary (web).
2. `INPUT_VALIDATION`: system validates mandatory fields (photo, name, age, interests, favorite color, favorite cartoons, hobbies).
   - **Elements:** Validation rules, error messages.
   - **User Action:** (None; validation runs on submit.)
   - **System Response/Trigger:** Validate; pass or show validation errors.
3. `INPUT_CONFIRMATION`: user confirms captured values.
   - **Elements:** Summary of captured values, Confirm/Edit actions.
   - **User Action:** Confirm or edit.
   - **System Response/Trigger:** On confirm, proceed; on edit, return to relevant input step.
4. `SERIES_GENERATION_REQUESTED`: system starts 5-series generation.
   - **Elements:** (Transition to generation state.)
   - **User Action:** (None.)
   - **System Response/Trigger:** Start 5-series generation.

#### Error States
- Invalid/missing age.
- Missing photo.
- Unsupported/corrupt photo upload.

#### Edge Cases
- User edits fields after summary review.
- Returning user changes only selected fields.

#### Exit Points
- Proceed to series options.

### Flow 5: Generate 5 series → Manual vs AI-Assisted selection → allow back/change
- **Goal:** Generate and select one series with explicit user decision and no timeout (P0-2, P0-3, P0-4).
- **Entry Point:** Validated child input capture.
- **Frequency:** Every generation attempt and series-regeneration round.

#### Happy Path
1. `SERIES_GENERATING`: system generates exactly 5 series and marks the first as Primary (recommended).
   - **Elements:** Progress indicator, generation state.
   - **User Action:** (None; wait for completion.)
   - **System Response/Trigger:** Generate 5 series; mark first as Primary; present options when ready.
2. `SERIES_OPTIONS_VIEW`: user sees 5 options with controls for Manual Selection, Auto Select, Back, Regenerate.
   - **Elements:** 5 series cards, Primary badge, Manual Selection, Auto Select, Back, Regenerate.
   - **User Action:** Choose Manual Selection (pick one) or Auto Select (AI-assisted).
   - **System Response/Trigger:** On Manual: store chosen series. On Auto Select: system selects one and shows short rationale; store selection.
3. User chooses:
   - Manual Selection: picks one series.
   - AI-Assisted Selection: taps Auto Select; system selects one and gives short rationale.
4. `SERIES_SELECTED`: system stores selected series + selection method + version metadata.
   - **Elements:** (Storage/state update.)
   - **User Action:** (Completed in step 2–3.)
   - **System Response/Trigger:** Persist selection; proceed to plot generation.
5. `PLOT_GENERATION_REQUESTED`: system proceeds to plot generation.
   - **Elements:** (Transition to plot flow.)
   - **User Action:** (None.)
   - **System Response/Trigger:** Start plot generation for selected series.

#### Error States
- Series generation failed.
- Selection persistence failed.

#### Edge Cases
- No user response → flow stays pending indefinitely (no timeout auto-selection).
- User goes back from plot stage and changes series.

#### Exit Points
- Continue to plot options.

### Flow 6: Generate 5 plots → Manual vs AI-Assisted selection → regeneration up to 3 rounds
- **Goal:** Select one plot with explicit decision and regeneration enforcement (P0-5, P0-6, P0-7).
- **Entry Point:** Selected series.
- **Frequency:** Every selected series path.

#### Happy Path
1. `PLOT_GENERATING`: system generates exactly 5 plots for selected series.
   - **Elements:** Progress indicator, generation state.
   - **User Action:** (None; wait for completion.)
   - **System Response/Trigger:** Generate 5 plots; present options when ready.
2. `PLOT_OPTIONS_VIEW`: user sees 5 plot options and controls (Manual Selection, Auto Select, Back to Series, Regenerate, round counter).
   - **Elements:** 5 plot options, Manual Selection, Auto Select, Back to Series, Regenerate, round counter.
   - **User Action:** Choose Manual Selection (pick one), Auto Select, Regenerate, or Back to Series.
   - **System Response/Trigger:** On Manual: store chosen plot. On Auto Select: system selects one and shows rationale; store. On Regenerate (if ≤3): new 5 plots. On Back: return to series options.
3. User chooses:
   - Manual Selection: picks one plot.
   - AI-Assisted Selection: taps Auto Select; system selects one and gives short rationale.
4. `PLOT_SELECTED`: system stores selected plot + selection method + version metadata.
   - **Elements:** (Storage/state update.)
   - **User Action:** (Completed in step 2–3.)
   - **System Response/Trigger:** Persist selection; proceed to preview generation.
5. `BOOK_GENERATION_REQUESTED`: system moves to preview generation.
   - **Elements:** (Transition to preview flow.)
   - **User Action:** (None.)
   - **System Response/Trigger:** Start preview (watermarked PDF) generation.

#### Error States
- Plot generation failed.
- Regeneration request beyond max rounds.

#### Edge Cases
- No user response → flow stays pending indefinitely.
- User returns to series selection and changes series.
- Regeneration reaches round 3 and is blocked thereafter.

#### Exit Points
- Continue to generation progress and preview.

### Flow 7: Generate watermarked preview PDF → preview viewing (web) + WhatsApp notification
- **Goal:** Ensure preview-before-payment flow (P0-12).
- **Entry Point:** Plot selected.
- **Frequency:** One per completed plot selection path.

#### Happy Path
1. `GENERATION_PROGRESS`: system shows generation progress state.
2. `PREVIEW_READY`: watermarked preview PDF is generated and stored.
3. `WEB_PREVIEW_VIEWER`: user views watermarked preview.
4. `WA_PREVIEW_NOTIFICATION`: WhatsApp confirms preview readiness with website link.

#### Error States
- Preview generation failed.
- Preview asset inaccessible.

#### Edge Cases
- User opens preview from multiple devices.
- User returns later and reopens preview from Book Detail.

#### Exit Points
- Proceed to checkout/payment.

### Flow 8: Purchase flow after preview
- **Goal:** Support paid unlock after preview for PDF-only or Print+Ship (P0-13, P0-14, P0-15). **Print+Ship is in scope for V1.**
- **Entry Point:** Preview Viewer or Book Detail.
- **Frequency:** For every preview intended for purchase.

#### Happy Path
1. `CHECKOUT_SELECTION`: user chooses PDF-only or Print+Ship.
   - **Elements:** PDF-only and Print+Ship options.
   - **User Action:** Select purchase type.
   - **System Response/Trigger:** Store choice; if Print+Ship, require shipping address next.
2. `SHIPPING_ADDRESS_CAPTURE` (Print+Ship only): user provides required address.
   - **Elements:** Address form (required for Print+Ship).
   - **User Action:** Enter shipping address and submit.
   - **System Response/Trigger:** Validate and store address; enable payment.
3. `PAYMENT_METHOD_SELECTION`: user selects PayPal or Razorpay.
   - **Elements:** PayPal and Razorpay options.
   - **User Action:** Select gateway and proceed.
   - **System Response/Trigger:** Redirect to selected payment gateway.
4. `PAYMENT_REDIRECT`: user completes payment and returns.
   - **Elements:** Gateway-hosted payment flow; return URL.
   - **User Action:** Complete payment at gateway; return to app.
   - **System Response/Trigger:** Receive callback; process result (success/failed/cancelled).
5. `PAYMENT_RESULT`:
   - Success: final PDF unlocked.
   - Print+Ship success: print order created with “Order Placed” minimum status.
6. `ORDER_CONFIRMATION`: user sees paid status and next actions.
   - **Elements:** Paid status, download CTA, print order status (if Print+Ship).
   - **User Action:** View confirmation; download or go to My Books.
   - **System Response/Trigger:** Show order confirmation and next actions.

#### Error States
- Payment failed.
- Payment cancelled.
- Payment success but unlock failed.
- Payment success but print order creation failed.

#### Edge Cases
- User exits payment flow and returns later to resume pending payment.
- Duplicate payment callback handling (TBD).

#### Exit Points
- Library/Book Detail with updated paid status.

### Flow 9: Delivery flow
- **Goal:** Deliver final access via email + website, and WhatsApp confirmation without direct PDF send (P0-14).
- **Entry Point:** Payment success.
- **Frequency:** Every paid order.

#### Happy Path
1. `DELIVERY_TRIGGERED`: final non-watermarked PDF is unlocked in website account.
2. `EMAIL_DELIVERY`: system sends final PDF email to registered user email.
3. `WA_CONFIRMATION_MESSAGE`: WhatsApp confirms payment and directs user to email + website link.
4. `LIBRARY_ACCESS`: user downloads/re-downloads from My Books/Book Detail.

#### Error States
- Email send failed.
- Unlock delay after payment success.

#### Edge Cases
- User asks to receive final PDF directly on WhatsApp → provide website link and email confirmation only.
- Updated email handling policy (TBD).

#### Exit Points
- Final PDF access complete.

### Flow 10: Returning user flow
- **Goal:** Enable repeat purchase flow with stored profiles, series, plots, and history (P0-16, P0-17, P1-1).
- **Entry Point:** Dashboard, My Children, or My Books.
- **Frequency:** Returning sessions.

#### Happy Path
1. `DASHBOARD_RETURNING`: user sees My Children/My Books totals and recent history.
   - **Elements:** Dashboard with counts, recent history, Start New Book.
   - **User Action:** (None; page loads.) Optionally navigate to My Children or My Books.
   - **System Response/Trigger:** Load and display account summary.
2. `CHILD_DETAIL`: user sees stored series (Primary tags), plot/session summaries, and completed books.
   - **Elements:** Child profile, series list, plot summaries, completed books.
   - **User Action:** Open child; choose Start new book (from existing series or new 5-series).
   - **System Response/Trigger:** Load child data; present start options.
3. User chooses:
   - Start new book from existing series (new plot generation), or
   - Start new 5-series generation.
   - **Elements:** Start-from-series and Start-new-5-series CTAs.
   - **User Action:** Select one path.
   - **System Response/Trigger:** Enter series/plot flow or input capture as applicable.
4. Standard flow continues through selection, preview, payment, and delivery.
   - **Elements:** Series options, plot options, preview, checkout, delivery (per other flows).
   - **User Action:** Complete selection, preview, payment.
   - **System Response/Trigger:** Same as Flows 5–9.
5. If selected plot duplicates a prior completed plot for same child, system shows warning before confirmation.
   - **Elements:** Duplicate-plot warning, Proceed or Change decision.
   - **User Action:** Proceed or change selection.
   - **System Response/Trigger:** On proceed: continue. On change: return to plot/series selection.

#### Error States
- History retrieval failed.
- Duplicate plot warning not resolved.

#### Edge Cases
- Multiple in-progress books for same child.
- User starts in WhatsApp and completes on website.

#### Exit Points
- New preview or paid order stored in history.

### Flow 11: Support flow
- **Goal:** Provide support, FAQ, and policy visibility.
- **Entry Point:** Public and authenticated navigation/footer.
- **Frequency:** As needed.

#### Happy Path
1. `SUPPORT_FAQ`: user browses FAQ/support topics.
2. `CONTACT_SUPPORT`: user submits support request.
3. `POLICIES`: user reviews Terms, Privacy, Refund/Shipping policies.
4. `SUPPORT_CONFIRMATION`: system confirms support submission.

#### Error States
- Support form submission failed.

#### Edge Cases
- User checks policy information before checkout.

#### Exit Points
- Return to dashboard, preview, checkout, or library.

### Flow 12: Error recovery flow
- **Goal:** Resume pending flows without data loss.
- **Entry Point:** User returns after interruption or failure.
- **Frequency:** Interruption scenarios.

#### Happy Path
1. Resume pending selection state with preserved options and regeneration count.
2. Resume pending payment with preserved purchase type and saved address draft (if applicable).
3. Resume failed generation state with retry while preserving selected series/plot.

#### Error States
- Pending session state missing.
- Pending state corrupted.

#### Edge Cases
- Cross-channel resume (WhatsApp ↔ Website).
- Multiple concurrent pending books.

#### Exit Points
- Continue from recovered stage to completion.

## 3. Navigation Map

### Website Navigation Tree
- **Public**
  - Home
  - About
  - Careers
  - Terms & Conditions
  - Privacy Policy
  - Refund/Shipping Policy
  - Stories Preview Gallery
  - Contact/Support
  - Login (Email + Password)
  - Sign Up (Email Registration)
  - Forgot Password
- **Authenticated**
  - Dashboard
  - My Children
    - Child Detail
    - Create/Edit Child
    - Create New Book
      - Series Options
      - Plot Options
      - Generation Progress
      - Preview Viewer
      - Checkout/Payment
      - Order Confirmation
  - My Books/Library
    - Book Detail
  - Account/Settings
  - Support/FAQ

### Cross-links
- Dashboard ↔ My Children / My Books.
- Child Detail → Create New Book / related Book Detail pages.
- Preview Viewer → Checkout/Payment.
- Order Confirmation → Book Detail / My Books.
- Support/FAQ and policy pages accessible from both public and authenticated contexts.

### WhatsApp Conversation State Map
- `WA_INIT`
  - `WA_ACCOUNT_MAPPING`
    - `WA_DASHBOARD_SUMMARY`
      - `WA_CHILD_SELECT_OR_CREATE`
        - `WA_INPUT_NAME`
          - `WA_INPUT_AGE`
            - `WA_INPUT_DETAILS`
              - `WA_INPUT_PHOTO`
                - `WA_INPUT_OPTIONAL_MORE_DETAILS`
                  - `WA_CONFIRM_INPUTS`
                    - `WA_SERIES_GENERATING`
                      - `WA_SERIES_OPTIONS`
                        - (Manual Selection or Auto Select)
                        - `WA_PLOT_GENERATING`
                          - `WA_PLOT_OPTIONS`
                            - (Manual Selection or Auto Select, Regenerate up to 3)
                            - `WA_PREVIEW_READY_NOTICE`
                              - `WEB_PREVIEW_VIEWER`
                                - `WEB_CHECKOUT_PAYMENT`
                                  - `WEB_ORDER_CONFIRMATION`
                                    - `WA_PAYMENT_CONFIRMATION`

### Navigation Rules
- Authentication via email + password is mandatory before accessing personalized data or purchase actions.
- Authentication is required for all authenticated website routes.
- WhatsApp account mapping is mandatory before child/book actions.
- Auto-selection occurs only when user taps Auto Select; there is no time-based auto-selection.
- User can navigate back from plots to series and change series.
- Payment is only available after watermarked preview exists.
- Final non-watermarked PDF access is only available after payment success.
- WhatsApp sends confirmation and website/email access details; it does not send final PDF file directly.

## 4. Screen Inventory

### Website Screens

#### `WEB_PUBLIC_HOME`
- **Route/State ID:** `WEB_PUBLIC_HOME`
- **Route:** /
- **Access:** Public
- **Purpose:** Marketing entry and authentication handoff.
- **Key Elements:** Hero CTA, Start on WhatsApp CTA, Login (Email + Password) CTA, Sign Up (Email Registration) CTA, public navigation links.
- **Actions → Destination:** Login → `WEB_AUTH_LOGIN`; Sign Up → `WEB_AUTH_SIGNUP`; Start on WhatsApp → `WA_INIT`.
- **State Variants:** Loading / Default / Content Error.

#### `WEB_AUTH_LOGIN`
- **Route/State ID:** `WEB_AUTH_LOGIN`
- **Route:** /login
- **Access:** Public
- **Purpose:** ApexNeural SaaS authentication (email + password).
- **Key Elements:** Email + Password login form, Forgot Password link.
- **Actions → Destination:** Success → `WEB_DASHBOARD`; Forgot Password → `WEB_AUTH_FORGOT_PASSWORD`; Failure → inline error.
- **State Variants:** Loading / Validation Error / Service Error.

#### `WEB_AUTH_SIGNUP`
- **Route/State ID:** `WEB_AUTH_SIGNUP`
- **Route:** /signup
- **Access:** Public
- **Purpose:** Email-based registration for new users.
- **Key Elements:** Email + Password signup form.
- **Actions → Destination:** Success → `WEB_DASHBOARD`; Failure → inline error (e.g. email already registered).
- **State Variants:** Loading / Validation Error / Service Error.

#### `WEB_AUTH_FORGOT_PASSWORD`
- **Route/State ID:** `WEB_AUTH_FORGOT_PASSWORD`
- **Route:** /forgot-password
- **Access:** Public
- **Purpose:** Request password reset link sent to email.
- **Key Elements:** Email input, submit for reset link.
- **Actions → Destination:** Submit → confirmation message; link in email → `WEB_AUTH_RESET_PASSWORD`.
- **State Variants:** Loading / Success / Error (e.g. reset link expired).

#### `WEB_AUTH_RESET_PASSWORD`
- **Route/State ID:** `WEB_AUTH_RESET_PASSWORD`
- **Route:** /reset-password
- **Access:** Public (via email reset link)
- **Purpose:** Set new password using reset link.
- **Key Elements:** New password fields, submit.
- **Actions → Destination:** Success → Login or Dashboard; Failure → inline error (e.g. reset link expired).
- **State Variants:** Loading / Validation Error / Success / Error.

#### `WEB_DASHBOARD`
- **Route/State ID:** `WEB_DASHBOARD`
- **Route:** /dashboard
- **Access:** Authenticated
- **Purpose:** User summary and primary actions.
- **Key Elements:** My Children count, My Books count, recent history, Start New Book CTA.
- **Actions → Destination:** My Children / My Books / New Book / Settings / Support.
- **State Variants:** Loading / Empty / Populated / Error.

#### `WEB_MY_CHILDREN`
- **Route/State ID:** `WEB_MY_CHILDREN`
- **Route:** /children
- **Access:** Authenticated
- **Purpose:** Child profile listing.
- **Key Elements:** Child cards, per-child book count, Create Child CTA.
- **Actions → Destination:** Open Child Detail / Create Child.
- **State Variants:** Loading / Empty / Error.

#### `WEB_CHILD_DETAIL`
- **Route/State ID:** `WEB_CHILD_DETAIL`
- **Route:** /children/:childId
- **Access:** Authenticated
- **Purpose:** Child-specific history and new-book start point.
- **Key Elements:** Stored series list, Primary tags, past plots/session summaries, completed books.
- **Actions → Destination:** New Book from existing series / New 5-series flow / Open related books.
- **State Variants:** Loading / Empty History / Error.

#### `WEB_CHILD_CREATE_EDIT`
- **Route/State ID:** `WEB_CHILD_CREATE_EDIT`
- **Route:** /children/:childId/edit
- **Access:** Authenticated
- **Purpose:** Create or edit child profile and inputs.
- **Key Elements:** Mandatory fields + optional “More details/favourites”.
- **Actions → Destination:** Save → Child Detail or flow continuation.
- **State Variants:** Loading / Validation Error / Save Success / Save Failure.

#### `WEB_SERIES_OPTIONS`
- **Route/State ID:** `WEB_SERIES_OPTIONS`
- **Route:** /books/:bookId/series
- **Access:** Authenticated
- **Purpose:** Present exactly 5 series and selection controls.
- **Key Elements:** 5 cards, Primary badge on first series, Manual Selection, Auto Select, Back, Regenerate.
- **Actions → Destination:** Select series / Auto Select / Back / Regenerate / Next to plot generation.
- **State Variants:** Generating / Ready / Pending / Error.

#### `WEB_PLOT_OPTIONS`
- **Route/State ID:** `WEB_PLOT_OPTIONS`
- **Route:** /books/:bookId/plots
- **Access:** Authenticated
- **Purpose:** Present exactly 5 plots and regeneration controls.
- **Key Elements:** 5 plot options, Manual Selection, Auto Select, Back to Series, Regenerate, Round counter.
- **Actions → Destination:** Select plot / Auto Select / Back / Regenerate / Next to generation progress.
- **State Variants:** Generating / Ready / Pending / Max Regen Reached / Error.

#### `WEB_GENERATION_PROGRESS`
- **Route/State ID:** `WEB_GENERATION_PROGRESS`
- **Route:** /books/:bookId/generating
- **Access:** Authenticated
- **Purpose:** Show generation progress before preview availability.
- **Key Elements:** Stage tracker and status text.
- **Actions → Destination:** Wait / Refresh / Continue when ready.
- **State Variants:** Loading / In Progress / Success / Failure.

#### `WEB_PREVIEW_VIEWER`
- **Route/State ID:** `WEB_PREVIEW_VIEWER`
- **Route:** /books/:bookId/preview
- **Access:** Authenticated
- **Purpose:** Watermarked preview viewing before payment.
- **Key Elements:** Watermarked PDF viewer, checkout CTA, regenerate CTA, back.
- **Actions → Destination:** Checkout / Regenerate (policy-limited) / Book Detail.
- **State Variants:** Loading / Ready / Error.

#### `WEB_CHECKOUT_PAYMENT`
- **Route/State ID:** `WEB_CHECKOUT_PAYMENT`
- **Route:** /books/:bookId/checkout
- **Access:** Authenticated
- **Purpose:** Purchase selection, address capture (if Print+Ship), and gateway payment.
- **Key Elements:** PDF-only vs Print+Ship selector, address form, PayPal/Razorpay options.
- **Actions → Destination:** Submit payment / Return to preview.
- **State Variants:** Loading / Validation Error / Payment Pending / Failed / Cancelled / Success.

#### `WEB_ORDER_CONFIRMATION`
- **Route/State ID:** `WEB_ORDER_CONFIRMATION`
- **Route:** /books/:bookId/confirmation
- **Access:** Authenticated
- **Purpose:** Confirm payment outcome and unlocked state.
- **Key Elements:** Paid status, download CTA, print order status.
- **Actions → Destination:** Download / Open My Books / Open Book Detail / Contact Support.
- **State Variants:** Success / Partial Failure / Error.

#### `WEB_MY_BOOKS_LIBRARY`
- **Route/State ID:** `WEB_MY_BOOKS_LIBRARY`
- **Route:** /books
- **Access:** Authenticated
- **Purpose:** View created/purchased books and statuses.
- **Key Elements:** Book list, status badges (Preview/Paid PDF/Paid Print), counts.
- **Actions → Destination:** Open Book Detail.
- **State Variants:** Loading / Empty / Error.

#### `WEB_BOOK_DETAIL`
- **Route/State ID:** `WEB_BOOK_DETAIL`
- **Route:** /books/:bookId
- **Access:** Authenticated
- **Purpose:** Manage single book lifecycle.
- **Key Elements:** Book status, preview link, checkout action, download/re-download, regenerate action.
- **Actions → Destination:** Preview / Checkout / Download / Regenerate / Support.
- **State Variants:** Preview-only / Paid-unlocked / Error.

#### `WEB_ACCOUNT_SETTINGS`
- **Route/State ID:** `WEB_ACCOUNT_SETTINGS`
- **Route:** /settings
- **Access:** Authenticated
- **Purpose:** Account preferences and profile settings.
- **Key Elements:** Account preference fields (TBD), Change Password option.
- **Actions → Destination:** Save settings; Change Password.
- **State Variants:** Loading / Save Success / Save Failure.

#### `WEB_SUPPORT_FAQ`
- **Route/State ID:** `WEB_SUPPORT_FAQ`
- **Route:** /support
- **Access:** Public + Authenticated
- **Purpose:** FAQ and support entry.
- **Key Elements:** FAQ list, support CTA, policy links.
- **Actions → Destination:** Contact support / policy pages.
- **State Variants:** Loading / Empty / Error.

#### `WEB_CONTACT_SUPPORT`
- **Route/State ID:** `WEB_CONTACT_SUPPORT`
- **Route:** /support/contact
- **Access:** Public + Authenticated
- **Purpose:** Submit support request.
- **Key Elements:** Support form and contact details.
- **Actions → Destination:** Submit ticket / back.
- **State Variants:** Loading / Validation Error / Success / Submission Failure.

#### `WEB_POLICIES`
- **Route/State ID:** `WEB_POLICIES`
- **Route:** /policies
- **Access:** Public + Authenticated
- **Purpose:** Show legal/policy documents.
- **Key Elements:** Terms, Privacy, Refund/Shipping policy content.
- **Actions → Destination:** Navigate back.
- **State Variants:** Loading / Error.

### WhatsApp States

#### `WA_INIT`
- **Route/State ID:** `WA_INIT`
- **Access:** Conversation initiated
- **Purpose:** Formal onboarding + Start CTA + photo consent note.
- **Key Elements:** Introductory message and Start action.
- **Actions → Destination:** Start → `WA_ACCOUNT_MAPPING`.
- **State Variants:** First-time / Returning.

#### `WA_ACCOUNT_MAPPING`
- **Route/State ID:** `WA_ACCOUNT_MAPPING`
- **Access:** Unmapped or mapping-required user
- **Purpose:** Map WhatsApp identity to email-based SaaS account (login with email + password if existing user; sign up with email + password if new user).
- **Key Elements:** Mapping prompts for email + password (login or signup).
- **Actions → Destination:** Success → `WA_DASHBOARD_SUMMARY`.
- **State Variants:** Pending / Success / Failure / Retry.

#### `WA_DASHBOARD_SUMMARY`
- **Route/State ID:** `WA_DASHBOARD_SUMMARY`
- **Access:** Mapped user
- **Purpose:** Show account summary and start options.
- **Key Elements:** My Children count, My Books count, quick actions.
- **Actions → Destination:** Child select/create / start book / website links.
- **State Variants:** Empty / Populated.

#### `WA_CHILD_SELECT_OR_CREATE`
- **Route/State ID:** `WA_CHILD_SELECT_OR_CREATE`
- **Access:** Mapped user
- **Purpose:** Child selection or profile creation.
- **Key Elements:** Existing child list and create option.
- **Actions → Destination:** Existing child flow or create child flow.
- **State Variants:** Single child / Multiple children / No children.

#### `WA_INPUT_NAME`
- **Route/State ID:** `WA_INPUT_NAME`
- **Access:** Mapped user with active creation flow
- **Purpose:** Capture child name first.
- **Key Elements:** Name prompt.
- **Actions → Destination:** Submit name → `WA_INPUT_AGE`.
- **State Variants:** Pending / Validation Error / Accepted.

#### `WA_INPUT_AGE`
- **Route/State ID:** `WA_INPUT_AGE`
- **Access:** Mapped user with active creation flow
- **Purpose:** Capture child age second.
- **Key Elements:** Age prompt.
- **Actions → Destination:** Submit age → `WA_INPUT_DETAILS`.
- **State Variants:** Pending / Validation Error / Accepted.

#### `WA_INPUT_DETAILS`
- **Route/State ID:** `WA_INPUT_DETAILS`
- **Access:** Mapped user with active creation flow
- **Purpose:** Capture interests, favorite color, favorite cartoons, hobbies.
- **Key Elements:** Structured detail prompts.
- **Actions → Destination:** Submit details → `WA_INPUT_PHOTO`.
- **State Variants:** Pending / Validation Error / Accepted.

#### `WA_INPUT_PHOTO`
- **Route/State ID:** `WA_INPUT_PHOTO`
- **Access:** Mapped user with active creation flow
- **Purpose:** Capture required child photo.
- **Key Elements:** Photo upload prompt and validation messages.
- **Actions → Destination:** Submit photo → `WA_INPUT_OPTIONAL_MORE_DETAILS`.
- **State Variants:** Pending / Validation Error / Accepted.

#### `WA_INPUT_OPTIONAL_MORE_DETAILS`
- **Route/State ID:** `WA_INPUT_OPTIONAL_MORE_DETAILS`
- **Access:** Mapped user with active creation flow
- **Purpose:** Optional free-text “More details/favourites”.
- **Key Elements:** Optional text prompt.
- **Actions → Destination:** Continue → `WA_CONFIRM_INPUTS`.
- **State Variants:** Skipped / Submitted.

#### `WA_CONFIRM_INPUTS`
- **Route/State ID:** `WA_CONFIRM_INPUTS`
- **Access:** Mapped user with active creation flow
- **Purpose:** Confirm collected inputs before generation.
- **Key Elements:** Input summary + confirmation prompt.
- **Actions → Destination:** Confirm → `WA_SERIES_GENERATING`; Edit → input states.
- **State Variants:** Pending / Confirmed.

#### `WA_SERIES_GENERATING`
- **Route/State ID:** `WA_SERIES_GENERATING`
- **Access:** Mapped user
- **Purpose:** Generate exactly 5 series.
- **Key Elements:** Generation status message.
- **Actions → Destination:** On completion → `WA_SERIES_OPTIONS`.
- **State Variants:** In Progress / Failed.

#### `WA_SERIES_OPTIONS`
- **Route/State ID:** `WA_SERIES_OPTIONS`
- **Access:** Mapped user
- **Purpose:** Present 5 series with Manual vs AI-Assisted selection.
- **Key Elements:** 5 options, Primary recommendation, Manual Selection, Auto Select, Regenerate, Back.
- **Actions → Destination:** Select/Auto/Regenerate/Back.
- **State Variants:** Ready / Pending / Error.

#### `WA_PLOT_GENERATING`
- **Route/State ID:** `WA_PLOT_GENERATING`
- **Access:** Mapped user
- **Purpose:** Generate exactly 5 plots.
- **Key Elements:** Generation status message.
- **Actions → Destination:** On completion → `WA_PLOT_OPTIONS`.
- **State Variants:** In Progress / Failed.

#### `WA_PLOT_OPTIONS`
- **Route/State ID:** `WA_PLOT_OPTIONS`
- **Access:** Mapped user
- **Purpose:** Present 5 plots with Manual vs AI-Assisted and regeneration.
- **Key Elements:** 5 plots, Manual Selection, Auto Select, Regenerate, Back to Series, round counter.
- **Actions → Destination:** Select/Auto/Regenerate/Back.
- **State Variants:** Ready / Pending / Max Regen Reached / Error.

#### `WA_PREVIEW_READY_NOTICE`
- **Route/State ID:** `WA_PREVIEW_READY_NOTICE`
- **Access:** Mapped user
- **Purpose:** Notify that preview is ready and provide website link.
- **Key Elements:** Preview-ready confirmation + secure URL.
- **Actions → Destination:** Open `WEB_PREVIEW_VIEWER`.
- **State Variants:** Sent / Send Failure.

#### `WA_PAYMENT_CONFIRMATION`
- **Route/State ID:** `WA_PAYMENT_CONFIRMATION`
- **Access:** Mapped user
- **Purpose:** Confirm payment status and direct to email + website for final PDF.
- **Key Elements:** Payment outcome message + website link.
- **Actions → Destination:** Open Book Detail / Contact Support.
- **State Variants:** Success / Failed / Cancelled.

## 5. Interaction Patterns

### Validation Rules

#### Authentication
- **Email:** Must match a valid email format (e.g. standard RFC 5322–style address; no leading/trailing spaces; trim on submit).
- **Password:** Minimum length 8 characters; must include at least one letter and one number (or meet defined complexity policy); no leading/trailing spaces.
- **Change Password / Reset:** New password must meet same criteria as above; current password required for change.

#### Age input
- **Format:** Numeric only; integer; no decimals.
- **Range:** Child age must be within defined range (e.g. 1–12 or 1–18); exact bounds TBD per product.
- Invalid or out-of-range value triggers validation error and blocks progression.

#### Photo upload
- **Supported types:** JPG, JPEG, PNG only.
- **File size:** Maximum file size per upload (e.g. 5 MB or 10 MB; TBD per product).
- **Integrity:** File must be non-corrupt and readable; unsupported type or corrupt file triggers validation error.
- **Suitability:** Photo must be suitable for personalization (e.g. clear, single child; detailed criteria TBD).
- Example message: "The uploaded photo could not be processed. Please upload a clear child photo in a supported image format (JPG or PNG)."

### Photo Upload Validation
- Mandatory photo before generation (P0-1).
- Validation includes: file presence, supported image type (JPG/PNG), non-corrupt file, basic quality suitability (TBD).
- Example message: “The uploaded photo could not be processed. Please upload a clear child photo in a supported image format.”

### 5-Option Selection UI + Manual vs Auto Select
- Exactly 5 options shown for series and plots.
- Two explicit actions: `Manual Selection` and `Auto Select (AI-Assisted)`.
- No automatic selection on inactivity.
- Pending state remains indefinitely until explicit user action.

### Regenerate Control + Counter (Max 3)
- Regenerate available in selection states.
- Counter displayed as `Regeneration Round X/3`.
- At max round, regenerate is blocked with clear message.
- Example message: “You have reached the maximum of 3 regeneration rounds for this stage. Please select one available option to continue.”

### Preview Watermark Viewer
- Preview remains watermarked pre-payment.
- Viewer displays clear preview status.
- Final non-watermarked download remains locked until payment success.

### Payment Redirect/Return Handling (PayPal/Razorpay)
- User is redirected to gateway and returned with outcome status.
- Outcome statuses: Success / Failed / Cancelled / Pending Verification.
- On non-success, user remains in checkout/preview with retry option.

### Email Delivery Confirmation + Resend Email
- On payment success, send final PDF email.
- Confirmation includes resend option.
- On resend failure, route user to support and keep website access available.

### Back Navigation + State Preservation
- Back from plots to series preserves valid state and regeneration count.
- Back from checkout preserves selected purchase type (and address draft when applicable).
- Re-entry resumes pending selection/payment/generation state.

## 6. Decision Points

```text
IF user selects Sign Up
THEN require email + password registration
  IF email already exists
    THEN show error
```

```text
IF user selects Forgot Password
THEN send reset link to email
```

```text
IF user opens an authenticated website route
THEN validate SaaS auth session
  IF valid session EXISTS
    THEN allow route
  ELSE redirect to Login (Email + Password) and preserve intended destination
```

```text
IF WhatsApp conversation starts
THEN check account mapping
  IF mapped = true
    THEN continue to child selection/dashboard summary
  ELSE require account mapping (login with email + password or sign up with email + password) before any book flow step
```

```text
IF existing child profiles are available
THEN show "Select Existing" and "Create New"
ELSE force create-new child flow
```

```text
IF user is at series or plot selection
THEN require explicit user decision:
  - Manual Selection
  - AI-Assisted Selection (Auto Select)
IF user does not choose
THEN keep flow pending indefinitely (no timeout auto-selection)
```

```text
IF user taps Regenerate
THEN increment regeneration_count
  IF regeneration_count <= 3
    THEN generate new options and store as new version
  ELSE block regenerate and request user selection from available options
```

```text
IF purchase type = PDF-only
THEN shipping address is NOT required
ELSE IF purchase type = Print+Ship
THEN shipping address IS required before payment submission
```

```text
IF payment status = success
THEN unlock final non-watermarked PDF
ELSE keep final PDF locked and present retry/cancel flow
```

```text
IF selected plot matches a previously used plot for the same child in a completed book
THEN show duplicate warning and require explicit proceed-or-change decision
```

```text
IF book status = Preview
THEN allow watermarked preview only and block final PDF download
ELSE IF book status = Paid/Unlocked
THEN allow final PDF download and re-download
```

## 7. Error Handling Flows

### 404 (Page Not Found)
- **Message:** “The page you’re looking for is unavailable. Please return to your dashboard or home page.”
- **Actions:** Go to Dashboard, Go to Home, Contact Support.
- **Recovery:** Restore nearest valid state where possible.
- **Logging:** Route, user/session reference, timestamp.

### 500 (Server Error)
- **Message:** “We’re unable to complete this action right now. Please try again in a moment.”
- **Actions:** Retry, Return to previous page, Contact Support.
- **Recovery:** Retry operation without discarding saved selections.
- **Logging:** Error code, flow stage, correlation ID, timestamp.

### Offline / Network Loss
- **Message:** “Connection lost. Please reconnect to continue your book creation.”
- **Actions:** Retry connection, Resume from saved step.
- **Recovery:** Restore pending flow state after reconnect.
- **Logging:** Network event + stage + timestamp.

### Authentication Errors
- **Invalid email/password:** "Invalid email or password. Please try again."
- **Email already registered:** "This email is already registered. Please log in or use a different email."
- **Password does not meet criteria:** "Password does not meet the required criteria. Please check and try again."
- **Reset link expired:** "This password reset link has expired. Please request a new one."
- **Password change failed:** "We could not update your password. Please try again or use Forgot Password."
- **Actions:** Correct input, retry, or use Forgot Password / request new reset link.
- **Recovery:** Return to Login, Sign Up, or Forgot Password as appropriate.
- **Logging:** Auth failure type, no credentials stored, timestamp.

### Invalid Inputs (name/age/photo)
- **Name Message:** “Please enter your child’s name to continue.”
- **Age Message:** “Please enter a valid child age to continue.”
- **Photo Message:** “Please upload a clear child photo in a supported image format.”
- **Actions:** Correct input and resubmit.
- **Recovery:** Preserve all previously valid inputs.
- **Logging:** Field-level validation failure and timestamp.

### Payment Failure/Cancel
- **Failure Message:** “Payment was not completed. Your preview is saved and you can try again.”
- **Cancel Message:** “Payment was cancelled. You can return anytime to complete your purchase.”
- **Actions:** Retry payment, switch gateway, return to preview.
- **Recovery:** Preserve purchase choice and address draft (if any).
- **Logging:** Gateway, status, order reference, timestamp.

### Generation Failure
- **Message:** “We could not generate your book preview at this moment. Please retry generation.”
- **Actions:** Retry generation, Contact Support.
- **Recovery:** Preserve selected series/plot and regeneration history.
- **Logging:** Failed stage (series/plot/preview), book/child reference, timestamp.

### Email Send Failure
- **Message:** “Your payment is confirmed, but we could not send the email yet. Your final PDF is available in My Books.”
- **Actions:** Resend Email, Open My Books, Contact Support.
- **Recovery:** Keep final PDF available through website download.
- **Logging:** Email failure reason, order reference, timestamp.

## 8. Responsive Behavior

### Mobile
- Compact navigation and CTA prioritization.
- Single-column series/plot option presentation.
- Vertical preview experience with zoom controls.
- Step-based checkout progression.

### Tablet
- Two-column layouts where suitable.
- Preview with optional side actions panel.
- Checkout supports summary + form visibility.

### Desktop
- Full navigation visibility.
- Multi-card grids for series/plot options.
- Wider preview viewport.
- Combined checkout summary, address (if needed), and payment panel.

## 9. Animations & Transitions

### Navigation Transitions
- Subtle fade/slide between major screens (150–250ms).

### Modals/Drawers
- Smooth transitions for confirmations (e.g., duplicate warning, max regeneration reached).

### Loading and Progress
- Professional progress indicator during generation stages.
- Stage labels update clearly (e.g., “Generating 5 plot options…”).

### Success and Error Feedback
- Brief success confirmation animation for save/selection/payment.
- Non-blocking error banners with clear recovery action.

