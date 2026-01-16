# Laravel Feature Plugin

A structured Laravel feature development workflow that combines Blueprint code generation, Filament admin resources, and the laravel-simplifier agent for rapid, consistent feature development.

## Overview

This plugin provides a systematic 7-phase approach to building Laravel features:

1. **Discovery & Validation** - Verify draft.yaml and understand what will be generated
2. **Blueprint Build** - Generate models, migrations, controllers from draft.yaml
3. **Database Migrations** - Apply migrations to create database tables
4. **Filament Resources** - Create admin CRUD interfaces for models
5. **Code Simplification** - Refine generated code using laravel-simplifier
6. **Tests** - Create feature and Filament tests
7. **Summary** - Document what was built

## Prerequisites

- Laravel Blueprint installed: `composer require --dev laravel-shift/blueprint`
- Filament v4 installed and configured
- laravel-simplifier plugin enabled
- A `draft.yaml` file in your project root

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add laravel/claude-code
```
```
/plugin install laravel-simplifier@laravel
```

## Usage

### Basic Usage

```bash
/laravel-feature
```

The command will guide you through the entire workflow interactively.

### With Feature Description

```bash
/laravel-feature Add blog posts with comments
```

## Workflow Details

### Phase 1: Discovery & Validation

Before running any commands, the skill:
- Verifies `draft.yaml` exists
- Parses the draft to identify models and controllers
- Presents a summary for your confirmation

### Phase 2: Blueprint Build

Runs `php artisan blueprint:build` to generate:
- Eloquent models with relationships
- Database migrations
- Controllers with resource methods
- Form request validation classes
- Model factories
- Database seeders

### Phase 3: Database Migrations

After confirmation, runs `php artisan migrate` to:
- Create new database tables
- Apply schema changes
- Set up foreign key constraints

### Phase 4: Filament Resources

For each new model, creates Filament admin resources:
- Resource class with form and table schemas
- List, Create, Edit pages
- Optional: Relation managers, widgets, custom pages

### Phase 5: Code Simplification

Launches the laravel-simplifier agent to refine all generated code:
- Add proper type declarations
- Apply PHP 8 features (constructor promotion, etc.)
- Ensure Laravel conventions
- Clean up redundant code

### Phase 6: Tests

Creates comprehensive tests:
- Feature tests for controller endpoints
- Filament Livewire tests for admin CRUD
- Uses model factories for test data

### Phase 7: Summary

Provides a complete summary:
- All files created
- Database changes made
- Admin URLs for new resources
- Suggested next steps

## Creating draft.yaml

The skill expects a `draft.yaml` file to exist. Here's the Blueprint syntax:

### Basic Model

```yaml
models:
  Post:
    title: string:400
    content: longtext
    published_at: nullable timestamp
    user_id: id foreign
```

### Model with Relationships

```yaml
models:
  Post:
    title: string:400
    content: longtext
    user_id: id foreign
    relationships:
      hasMany: Comment
      belongsTo: User
      belongsToMany: Tag
```

### Resource Controller

```yaml
controllers:
  Post:
    resource
```

### Custom Controller Actions

```yaml
controllers:
  Post:
    index:
      query: all:posts
      render: post.index with:posts
    store:
      validate: post
      save: post
      redirect: post.index
```

### With Seeders

```yaml
models:
  Post:
    title: string
    content: text

seeders: Post
```

## Example Complete draft.yaml

```yaml
models:
  Category:
    name: string:100
    slug: string:100 unique
    description: nullable text
    relationships:
      hasMany: Post

  Post:
    title: string:400
    slug: string:400 unique
    content: longtext
    excerpt: nullable text
    published_at: nullable timestamp
    category_id: id foreign
    user_id: id foreign
    relationships:
      belongsTo: Category, User
      hasMany: Comment
      belongsToMany: Tag

  Comment:
    content: text
    post_id: id foreign
    user_id: id foreign
    relationships:
      belongsTo: Post, User

  Tag:
    name: string:50
    slug: string:50 unique
    relationships:
      belongsToMany: Post

controllers:
  Category:
    resource: api

  Post:
    resource

  Comment:
    resource: api.index, api.store, api.destroy

seeders: Category, Post, Comment, Tag
```

## Tips

1. **Start simple**: Begin with models and basic resource controllers
2. **Use relationships**: Blueprint generates proper Eloquent relationships
3. **Review before migrating**: Check the generated migrations before running
4. **Customize after generation**: The generated code is a starting point
5. **Run tests**: Always verify with `php artisan test` after generation

## Troubleshooting

### "draft.yaml not found"

Create a draft.yaml in your project root, or run:
```bash
php artisan blueprint:new
```

### Blueprint build errors

- Check YAML syntax (proper indentation)
- Model names should be StudlyCase singular
- Column types must be valid Laravel migration types

### Migration errors

- Check foreign key ordering (referenced tables must exist)
- Look for duplicate column/table names
- Use `php artisan migrate:rollback` to undo

### Filament resource errors

- Ensure model exists in app/Models
- Check if resource already exists
- Verify Filament panel is configured

## Related Tools

- [Laravel Blueprint](https://github.com/laravel-shift/blueprint) - Code generation from YAML
- [Filament](https://filamentphp.com/) - Admin panel framework
- [laravel-simplifier](../../../..) - Code simplification agent

## Author

Kaikei Development Team

## Version

1.0.0
