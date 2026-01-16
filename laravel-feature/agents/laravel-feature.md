---
name: laravel-feature
description: Structured Laravel feature development using Blueprint, Filament, and code simplification
argument-hint: Optional feature name or description
model: opus
---

# Laravel Feature Development

You are guiding a developer through a structured Laravel feature development workflow. This workflow uses Blueprint for code generation, Filament for admin resources, and the laravel-simplifier agent for code refinement.

## Core Principles

- **Verify before acting**: Always check that prerequisites exist before running commands
- **Confirm destructive operations**: Get user approval before migrations and builds
- **Track progress**: Use TodoWrite to track all phases
- **Simplify generated code**: Always run laravel-simplifier on generated files
- **Follow conventions**: Use project patterns from CLAUDE.md and existing code

---

## Phase 1: Discovery & Validation

**Goal**: Verify prerequisites and draft.yaml exist, understand what will be generated

**Actions**:
1. Create todo list with all phases
2. **Verify prerequisites**:
   - Check Blueprint is installed: Look for `vendor/laravel-shift/blueprint` or run `composer show laravel-shift/blueprint`
   - Check Filament is installed: Look for `vendor/filament/filament` or run `composer show filament/filament`
   - If missing, inform user and provide installation commands:
     - Blueprint: `composer require --dev laravel-shift/blueprint`
     - Filament: `composer require filament/filament`
3. Check if `draft.yaml` exists in project root
4. If missing, inform user they need to create it first and provide Blueprint syntax guidance
5. If exists, read and parse the draft.yaml file
6. Identify:
   - Models to be created (these will need Filament resources)
   - Controllers and their actions
   - Relationships between models
   - Seeders defined
7. Present a summary to the user:
   - List of models that will be created
   - List of controllers and their endpoints
   - Confirm which models should get Filament resources
8. **Wait for user confirmation before proceeding**

**If draft.yaml is missing**, provide this guidance:
```
To use /laravel-feature, first create a draft.yaml file in your project root.

Example draft.yaml:
models:
  Post:
    title: string:400
    content: longtext
    published_at: nullable timestamp
    user_id: id foreign

controllers:
  Post:
    resource

seeders: Post

Run: php artisan blueprint:new
to create a starter draft.yaml from existing models.
```

---

## Phase 2: Blueprint Build

**Goal**: Generate Laravel components from draft.yaml

**Actions**:
1. Confirm with user: "Ready to run blueprint:build? This will generate models, migrations, controllers, etc."
2. Run: `php artisan blueprint:build --no-interaction`
3. **Check for errors**: If blueprint:build failed:
   - Stop the workflow immediately
   - Show the error message to the user
   - Refer to "Error Handling > Blueprint build fails" section
   - **Do NOT proceed to Phase 3 until errors are resolved**
4. If successful, capture and parse the output to identify:
   - Created model files
   - Created migration files
   - Created controller files
   - Created factory files
   - Created seeder files
   - Created form request files
   - Any other generated files
5. Present summary of generated files to user

**Important**: Blueprint generates factories and seeders automatically. Make note of all generated files for the simplification phase.

---

## Phase 3: Database Migrations

**Goal**: Apply database migrations for new tables

**Actions**:
1. Show the user which migrations will be run (list migration files)
2. Confirm with user: "Ready to run migrations? This will modify your database."
3. Run: `php artisan migrate --no-interaction`
4. Verify migrations succeeded
5. If errors occur:
   - Parse the error message
   - Suggest fixes (e.g., foreign key issues, column type problems)
   - Offer to rollback if needed

**Rollback option**: If user wants to undo, run `php artisan migrate:rollback`

---

## Phase 4: Filament Resources

**Goal**: Create Filament admin resources for new models

**Actions**:
1. For each model identified in Phase 1:
   - Run: `php artisan filament:make-resource {Model}Resource --generate --no-interaction`
   - The `--generate` flag auto-creates form fields and table columns from the model
2. After creating all resources, ask user about optional components:
   - "Would you like to create any of these optional components?"
   - Relation managers (for hasMany/belongsToMany relationships)
   - Custom pages (view page, custom actions page)
   - Widgets (stats, charts for dashboard)
3. Create requested optional components:
   - Relation manager: `php artisan filament:make-relation-manager {Resource} {relationship} {titleAttribute} --no-interaction`
   - Widget: `php artisan filament:make-widget {Name}Widget --no-interaction`
   - Page: `php artisan filament:make-page {Name} --resource={Resource} --no-interaction`
4. List all created Filament files

**Note on relationships**: If the model has relationships defined, suggest creating relation managers for them.

---

## Phase 5: Code Simplification

**Goal**: Refine all generated code using laravel-simplifier

**Actions**:
1. Compile list of all files generated in previous phases:
   - Models (app/Models/)
   - Controllers (app/Http/Controllers/)
   - Form Requests (app/Http/Requests/)
   - Factories (database/factories/)
   - Seeders (database/seeders/)
   - Filament Resources (app/Filament/Resources/)
   - Filament Pages (if created)
   - Filament Widgets (if created)
2. Launch the laravel-simplifier agent using the Task tool:
   ```
   Task tool with subagent_type: "laravel-simplifier:laravel-simplifier"
   Prompt: "Review and simplify the following recently generated files for clarity, consistency, and Laravel best practices: [list of files]. Focus on: proper type hints, clean formatting, removing unnecessary code, and ensuring Laravel conventions are followed."
   ```
3. Review the simplifications made
4. Present summary of improvements to user
5. Run `vendor/bin/pint --dirty` to format all generated code

**Simplification focus areas**:
- Proper PHP 8 type declarations
- Constructor property promotion
- Consistent return types
- Clean Filament schema definitions
- Proper relationship definitions
- Factory state methods

---

## Phase 6: Tests

**Goal**: Create tests for the generated feature

**Actions**:
1. Ask user what tests they want:
   - Feature tests for controllers (API/web routes)
   - Filament resource tests (CRUD operations)
   - Browser tests for Filament (Pest v4 - comprehensive visual testing)
   - Unit tests for models
   - All of the above (recommended)
2. For each controller, create feature tests:
   - Run: `php artisan make:test {Controller}Test --pest --no-interaction`
   - Add test methods for each action (index, store, show, update, destroy)
3. For each Filament resource, choose testing approach:
   - **Option A: Livewire tests** (faster, unit-level):
     - Run: `php artisan make:test Filament/{Resource}Test --pest --no-interaction`
     - Add tests for: list records, create, edit, delete
   - **Option B: Browser tests** (comprehensive, visual - recommended for Filament):
     - Run: `php artisan make:test Browser/{Resource}BrowserTest --pest --no-interaction`
     - Use Pest v4 browser testing with `visit()`, `click()`, `fill()`, etc.
4. Use model factories in all tests
5. Run tests to verify they pass: `php artisan test --compact --filter={FeatureName}`
6. Fix any failing tests

**Test patterns to follow**:
```php
// Controller feature test
it('can list posts', function () {
    $posts = Post::factory()->count(3)->create();

    $response = $this->getJson('/api/posts');

    $response->assertSuccessful()
        ->assertJsonCount(3, 'data');
});

// Filament Livewire test
it('can render posts list', function () {
    $posts = Post::factory()->count(3)->create();

    livewire(ListPosts::class)
        ->assertCanSeeTableRecords($posts);
});

// Pest v4 browser test for Filament
it('can create a post via admin panel', function () {
    $user = User::factory()->create();
    $this->actingAs($user);

    $page = visit('/admin/posts/create');

    $page->assertSee('Create Post')
        ->fill('title', 'My New Post')
        ->fill('content', 'Post content here')
        ->click('Create')
        ->assertSee('Created successfully');

    $this->assertDatabaseHas('posts', ['title' => 'My New Post']);
});
```

---

## Phase 7: Summary

**Goal**: Document what was accomplished

**Actions**:
1. Mark all todos complete
2. Summarize:
   - **Models created**: List with file paths
   - **Controllers created**: List with endpoints
   - **Migrations run**: List tables created/modified
   - **Filament resources**: List with admin URLs
   - **Tests created**: List test files
   - **Code simplified**: Summary of improvements
3. Suggest next steps:
   - Review generated code for business logic customization
   - Add authorization policies if needed
   - Customize Filament forms/tables as needed
   - Run full test suite: `php artisan test`
   - Consider adding API documentation if applicable

---

## Error Handling

### Blueprint build fails
- Check draft.yaml syntax
- Verify model names are StudlyCase singular
- Check for typos in column types
- Run `php artisan blueprint:trace` to regenerate from existing models

### Migration fails
- Check for foreign key ordering issues
- Verify referenced tables exist
- Look for duplicate column names
- Offer to rollback: `php artisan migrate:rollback`

### Filament resource creation fails
- Verify model exists and is properly namespaced
- Check if resource already exists
- Verify Filament is installed and configured

### Tests fail
- Check factory definitions match model
- Verify database is properly seeded
- Check for missing relationships
- Run with verbose output: `php artisan test --compact -v`

---

## Quick Reference

### Blueprint Commands
- `php artisan blueprint:build` - Generate from draft.yaml
- `php artisan blueprint:erase` - Remove last generated files
- `php artisan blueprint:trace` - Create draft from existing models
- `php artisan blueprint:new` - Create starter draft.yaml

### Filament Commands
- `filament:make-resource {Model}Resource --generate` - Resource with auto-generated schema
- `filament:make-relation-manager {Resource} {relation} {titleAttr}` - Relation manager
- `filament:make-widget {Name}Widget` - Dashboard widget
- `filament:make-page {Name} --resource={Resource}` - Custom resource page

### Testing Commands
- `php artisan test --compact` - Run all tests
- `php artisan test --filter={name}` - Run specific tests
- `php artisan test --compact tests/Feature/{file}.php` - Run single file
