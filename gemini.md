# Gemini Work Log

## Couples Budget Planner Integration into Firefly III

This log details the steps taken to integrate a "Couples Budget Planner" (originally `app.html`) into the Firefly III personal finance management application.

### Phase 1: Initial Setup and Data Loading

*   **Cloned Firefly III Repository:** The Firefly III open-source project was cloned into the local environment.
*   **New Web Route (`/couples`):** A new web route was added to Firefly III's `routes/web.php` to serve the Couples Budget Planner.
*   **CouplesController Creation:** A new Laravel controller (`FireflyIII\Http\Controllers\CouplesController`) was created to handle requests for the planner page.
*   **Twig View Integration:** The HTML, CSS, and JavaScript from the original `app.html` were integrated into a new Twig template (`resources/views/couples/index.twig`), extending Firefly III's default layout. Twig's `{% raw %}` and `{% endraw %}` tags were used to prevent conflicts with JavaScript.
*   **Navigation Link:** A new link to the Couples Budget Planner was added to Firefly III's main sidebar navigation (`resources/views/partials/menu-sidebar.twig`).
*   **API Endpoint for State (`GET /api/v1/couples/state`):** A new API endpoint was created in `routes/api.php` to provide the initial state data for the planner.
*   **API Controller for State:** The `FireflyIII\Api\V1\Controllers\Couples\CouplesController` was implemented to fetch data from the Firefly III database, including:
    *   Authenticated user's name and calculated income for the current month (from revenue accounts).
    *   Expenses categorized by tags (`couple-p1`, `couple-p2`, `couple-shared`) for personal and shared expenses.
    *   Unassigned expenses (transactions without couple-related tags).
    *   Goals (mapped from Firefly III's "Piggy Banks").
*   **Frontend State Loading:** The frontend JavaScript was refactored to fetch its initial state from this new API endpoint, replacing the original `localStorage` persistence.

### Phase 2: Core CRUD Operations and UI Interactivity

*   **Transaction Creation (`POST /api/v1/couples/transactions`):**
    *   A new API endpoint was added to handle the creation of new transactions.
    *   The `storeTransaction` method in `CouplesController` was implemented to create `TransactionJournal` and `Transaction` records, assigning appropriate `couple-` tags based on the input column.
    *   The frontend's `handleFormSubmit` function was refactored to call this API endpoint.
*   **Transaction Update (`PUT /api/v1/couples/transactions/{transaction}`):**
    *   A new API endpoint was added for updating existing transactions.
    *   The `updateTransaction` method in `CouplesController` was implemented to modify `TransactionJournal` and `Transaction` details.
    *   The frontend's `handleListInteraction` function was refactored to call this API endpoint when transaction details are edited.
*   **Transaction Deletion (`DELETE /api/v1/couples/transactions/{transaction}`):**
    *   A new API endpoint was added for deleting transactions.
    *   The `deleteTransaction` method in `CouplesController` was implemented to soft-delete `TransactionJournal` records.
    *   The frontend's `handleListInteraction` function was refactored to call this API endpoint when the delete button is clicked.
*   **Transaction Tag Update (Drag-and-Drop) (`PUT /api/v1/couples/transactions/{transaction}/tag`):**
    *   A new API endpoint was added to update the tags of a transaction.
    *   The `updateTransactionTag` method in `CouplesController` was implemented to remove existing `couple-` tags and attach a new one based on the target column.
    *   The frontend's `handleDrop` function was refactored to call this API endpoint for drag-and-drop operations.
*   **Goal Creation (`POST /api/v1/couples/goals`):**
    *   A new API endpoint was added for creating new goals (piggy banks).
    *   The `storeGoal` method in `CouplesController` was implemented to create `PiggyBank` records.
    *   The frontend's `addGoal` function was refactored to call this API endpoint.
*   **Goal Deletion (`DELETE /api/v1/couples/goals/{goal}`):**
    *   A new API endpoint was added for deleting goals.
    *   The `deleteGoal` method in `CouplesController` was implemented to delete `PiggyBank` records.
    *   The frontend's `removeGoal` function was refactored to call this API endpoint.
*   **Tailwind CSS Integration:**
    *   Tailwind CSS dependencies were added to `resources/assets/v2/package.json`.
    *   `tailwind.config.js` and `postcss.config.js` were created in `resources/assets/v2`.
    *   Tailwind directives were added to `resources/assets/v2/src/sass/app.scss`.

### Remaining Tasks (Future Work)

*   **Person 2 Configuration:** Implement a feature to allow the user to select another Firefly III user as their partner, and dynamically fetch their income and expenses.
### Context for "Person 2 Configuration"

To implement the "Person 2 Configuration" feature, leveraging Firefly III's existing multi-user capabilities through `UserGroup`s appears to be the most idiomatic approach.

**Key Findings from Code Review:**

*   **`User` Model (`app/User.php`):**
    *   Users belong to a `UserGroup` (`userGroup()` relationship).
    *   Users have various relationships to financial entities (`accounts()`, `transactions()`, `piggyBanks()`), indicating individual ownership.
    *   Methods like `hasRoleInGroupOrOwner` suggest a robust permission system tied to `UserGroup`s.
    *   No direct "partner" relationship exists, implying a custom solution or leveraging existing group structures.
    *   `preferences()` relationship could store a partner's user ID if a custom preference-based solution is chosen.
*   **`UserGroup` Model (`app/Models/UserGroup.php`):**
    *   A `UserGroup` has a `title`.
    *   Crucially, a `UserGroup` has `HasMany` relationships to many core financial entities (e.g., `Account`, `Bill`, `Budget`, `Category`, `PiggyBank`, `TransactionJournal`). This means financial data can be *owned by a group*, not just individual users.
    *   `groupMemberships()` links `UserGroup`s to `User`s and `UserRole`s, defining user participation and permissions within the group.

**Proposed Implementation Strategy using UserGroups:**

1.  **User Interface for Partner Selection:**
    *   Add a new input field (e.g., a dropdown or autocomplete) in the "Settings" tab of the Couples Budget Planner.
    *   This field will allow the current user to select another user as their "partner."
    *   The selection should ideally be limited to users who are members of the *same* `UserGroup` as the authenticated user.
    *   A new API endpoint will be needed to save this selected partner's `user_id` as a user preference.
2.  **API Data Fetching for Partner:**
    *   Modify the `CouplesController@state` method.
    *   If a partner `user_id` is set in the user's preferences, fetch their income and expenses. This will involve querying for the partner's data (income from their revenue accounts, expenses from transactions tagged with `couple-p2` or `couple-shared` if they are also contributing to shared expenses).
    *   The `person2` name in the frontend state will be the partner's actual name.
    *   The `person2` income will be the partner's calculated income.
    *   The `person2` transactions will be the partner's transactions (potentially tagged `couple-p2`).
3.  **Transaction Management for Partner:**
    *   When creating, updating, or deleting transactions that are attributed to `person2` (or shared expenses), the API endpoints (`storeTransaction`, `updateTransaction`, `deleteTransaction`, `updateTransactionTag`) will need to be modified.
    *   These modifications will ensure that the transactions are correctly associated with the partner's `user_id` and the appropriate accounts/tags.

**Considerations:**

*   **Permissions:** Carefully manage permissions to ensure the authenticated user has the necessary rights to view and modify the partner's data within the `UserGroup`. Firefly III's `UserRoleEnum` (e.g., `READ_ONLY`, `MEMBER`, `OWNER`, `FULL`) will be crucial for defining these access levels.
*   **Data Ownership:** When new transactions are created for the partner, they should be owned by the partner's `user_id` in the database.
*   **UI for Partner Selection:** The frontend UI for selecting a partner could use an autocomplete field that dynamically searches for users within the same `UserGroup` to simplify the user experience.