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

### Phase 3: Person 2 Configuration

*   **Partner Selection UI:** Added a dropdown to the "Settings" tab of the Couples Budget Planner to allow selecting a partner user.
*   **API for User Listing (`GET /api/v1/couples/users`):** Implemented an API endpoint to fetch users within the authenticated user's `UserGroup` for partner selection.
*   **API for Partner Preference Saving (`POST /api/v1/couples/partner`):** Implemented an API endpoint to save the selected partner's user ID as a user preference.
*   **Dynamic Partner Data Fetching:** Modified the `CouplesController@state` method to fetch and include the partner's name, income, and transactions (tagged `couple-p2`) in the state response if a partner is selected.
*   **Frontend Partner Selection:** Updated the frontend JavaScript to populate the partner dropdown dynamically and pre-select the saved partner.

### Docker and Supabase Integration

*   **Supabase Local Setup:** Firefly III is configured to connect to a locally running Supabase instance for its database.
    *   **User Action Required:** To run Firefly III, you must first install the Supabase CLI and start the local Supabase stack.
        ```bash
        npm install -g supabase
        supabase init
        supabase start
        ```
    *   The default local Supabase PostgreSQL connection details are:
        *   **DB URL:** `postgresql://postgres:postgres@localhost:54322/postgres`
        *   **Username:** `postgres`
        *   **Password:** `postgres`
*   **Firefly III Configuration for Supabase:**
    *   The `.env` file was updated to use PostgreSQL and connect to the local Supabase instance:
        ```
        DB_CONNECTION=pgsql
        DB_HOST=localhost
        DB_PORT=54322
        DB_DATABASE=postgres
        DB_USERNAME=postgres
        DB_PASSWORD=postgres
        ```
    *   The `docker-compose.yml` file was modified to remove its internal MySQL database service, as Firefly III will now connect to the external Supabase instance.
    *   The `.db.env` file was deleted as it is no longer needed.