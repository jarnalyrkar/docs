---
title: 'GraphQL API'
intro: 'The GraphQL API is a **read-only** API for delivering content from Statamic to your frontend, external apps, SPAs, and numerous other possible sources. Content is delivered as JSON data.'
pro: true
blueprint: page
id: fc564ddf-80c1-4d87-8675-4a41f13c7774
---
(If you're interested in a [REST API](/content-api), we have one of those too.)

## Enable GraphQL

Enable the GraphQL API in your config or with an environment variable.

```php
// config/statamic/graphql.php
'enabled' => true,
```

```env
STATAMIC_GRAPHQL_ENABLED=true
```

:::tip
If you publish the underlying [package's](#laravel-package) config, the query routes will be enabled regardless of whether you've disabled it in the Statamic config.
:::

You will also need to enable the resources you want to be available. For security, they're all disabled by default.

```php
// config/statamic/graphql.php

'resources' => [
  'collections' => true,
  'taxonomies' => true,
  // etc
]
```

## Interfaces

Statamic will provide "interface" types, which describe more generic items. For instance, an `EntryInterface` exists for all
entries, which would provide fields like `id`, `slug`, `status`, `title`, and so on.

In addition to the interfaces, Statamic will provide implementations of them, which would come from the blueprints.

For example, if you had a collection named `pages`, and it had blueprints of `page` and `home`, you would find `Entry_Pages_Page`
and `Entry_Pages_Home` types. These implementations would provide fields specific to the blueprint, like `subtitle`, `content`, etc.

```graphql
{
    entries {
        id
        title
        ... on Entry_Pages_Page {
            subtitle
            content
        }
        ... on Entry_Pages_Home {
            hero_intro
            hero_image
        }
    }
}
```

## Queries

Statamic has a number of root level queries you can perform to get data.

You can read about the [available queries](#available-queries) further down the page,
but know that you can perform more than one query at a time. They just need to be at the top level of your GraphQL query body.

For example, the following would perform both `entries` and `collections` queries

```graphql
{
    entries {
        # ...
    }
    collections {
        # ...
    }
}
```

The response will contain the results of both queries:

```json
{
    "entries": { /* ... */ },
    "collections": { /* ... */ },
}
```

Note that you can even perform the same query multiple times. If you want to do this, you should use aliases:

```graphql
{
    home: entry(id: "home") {
        title
    }
    contact: entry(id: "contact") {
        title
    }
}
```

```json
{
    "home": { /* ... */ },
    "contact": { /* ... */ },
}
```

## Available Queries

- [Ping](#ping-query)
- [Collections](#collections-query)
- [Collection](#collection-query)
- [Entries](#entries-query)
- [Entry](#entry-query)
- [Asset Containers](#asset-containers-query)
- [Asset Container](#asset-container-query)
- [Assets](#assets-query)
- [Asset](#asset-query)
- [Taxonomies](#taxonomies-query)
- [Taxonomy](#taxonomy-query)
- [Terms](#terms-query)
- [Term](#term-query)
- [Global Sets](#global-sets-query)
- [Global Set](#global-set-query)
- [Navs](#navs-query)
- [Nav](#nav-query)

### Ping {#ping-query}

Used for testing that your connection works. If you send a query of `{ping}`, you should receive `{"data": {"ping": "pong"}}`.

```graphql
{
    ping
}
```

```json
{
    "data": {
        "ping": "pong"
    }
}
```

### Collections {#collections-query}

Used for querying collections.

Returns a list of [Collection](#collection-type) types.

```graphql
{
    collections {
        handle
        title
    }
}
```

```json
{
    "collections": [
        { "handle": "blog", "title": "Blog Posts" },
        { "handle": "events", "title": "Events" },
    ]
}
```

### Collection {#collection-query}

Used for querying a single collection.

Returns a [Collection](#collection-type) type.

```graphql
{
    collection(handle: "blog") {
        handle
        title
    }
}
```

```json
{
    "collections": {
        "handle": "blog",
        "title": "Blog Posts"
    }
}
```

### Entries {#entries-query}

Used for querying multiple entries.

Returns a [paginated](#pagination) list of [EntryInterface](#entry-interface) types.

| Argument | Type | Description |
|----------|------|-------------|
| `collection` | `[String]` | Narrows down the results by entries in one or more collections.
| `limit` | `Int` | The number of results to be shown per paginated page.
| `page` | `Int` | The paginated page to be shown. Defaults to `1`.
| `filter` | `JsonArgument` | Narrows down the results based on [filters](#filtering).
| `sort` | `[String]` | [Sorts](#sorting) the results based on one or more fields and directions.

Example query and response:

```graphql
{
    entries {
        current_page
        data {
            id
            title
        }
    }
}
```

```json
{
    "entries": {
        "current_page": 1,
        "data": [
            { "id": 1, "title": "First Entry" },
            { "id": 2, "title": "Second Entry" }
        ]
    }
}
```

### Entry {#entry-query}

Used for querying a single entry.

```graphql
{
    entry(id: 1) {
        id
        title
    }
}
```

```json
{
    "entry": {
        "id": 1,
        "title": "First Entry"
    }
}
```

### Asset Containers {#asset-containers-query}

Used for querying asset containers.

```graphql
{
    assetContainers {
        handle
        title
    }
}
```

```json
{
    "assetContainers": [
        { "handle": "images", "title": "Images" },
        { "handle": "documents", "title": "Documents" },
    ]
}
```

### Asset Container {#asset-container-query}

Used for querying a single asset container.

Returns an [AssetContainer](#asset-container-type) type.

```graphql
{
    assetContainer(handle: "images") {
        handle
        title
    }
}
```

```json
{
    "assetContainer": {
        "handle": "images",
        "title": "Images"
    }
}
```

| Argument | Type | Description |
|----------|------|-------------|
| `handle` | `String!` | Specifies which asset container to retrieve.

### Assets {#assets-query}

Used for querying multiple assets of an asset container.

Returns a [paginated](#pagination) list of [AssetInterface](#asset-interface) types.

| Argument | Type | Description |
|----------|------|-------------|
| `container` | `String!` | Specifies which asset container to query.
| `limit` | `Int` | The number of results to be shown per paginated page.
| `page` | `Int` | The paginated page to be shown. Defaults to `1`.
| `sort` | `[String]` | [Sorts](#sorting) the results based on one or more fields and directions.

Example query and response:

```graphql
{
    assets(container: "images") {
        current_page
        data {
            url
        }
    }
}
```

```json
{
    "entries": {
        "current_page": 1,
        "data": [
            { "url": "/assets/images/001.jpg" },
            { "url": "/assets/images/002.jpg" },
        ]
    }
}
```

### Asset {#asset-query}

Used for querying a single asset.

```graphql
{
    asset(id: 1) {
        id
        title
    }
}
```

```json
{
    "asset": {
        "id": 1,
        "title": "First Entry"
    }
}
```

You can either query by `id`, or by `container` and `path` together.

| Argument | Type | Description |
|----------|------|-------------|
| `id` | `String` | The ID of the asset. If you use this, you don't need `container` or `path`.
| `container` | `String` | The container to look for the asset. You must also provide the `path`.
| `path` | `String` | The path to the asset, relative to the container. You must also provide the `container`.

### Taxonomies {#taxonomies-query}

Used for querying taxonomies.

```graphql
{
    taxonomies {
        handle
        title
    }
}
```

```json
{
    "taxonomies": [
        { "handle": "tags", "title": "Tags" },
        { "handle": "categories", "title": "Categories" },
    ]
}
```

### Taxonomy {#taxonomy-query}

Used for querying a single taxonomy.

```graphql
{
    taxonomy(handle: "tags") {
        handle
        title
    }
}
```

```json
{
    "taxonomy": {
        "handle": "tags",
        "title": "Tags"
    }
}
```

### Terms {#terms-query}

Used for querying multiple taxonomy terms.

Returns a [paginated](#pagination) list of [TermInterface](#term-interface) types.

| Argument | Type | Description |
|----------|------|-------------|
| `taxonomy` | `[String]` | Narrows down the results by terms in one or more taxonomies.
| `limit` | `Int` | The number of results to be shown per paginated page.
| `page` | `Int` | The paginated page to be shown. Defaults to `1`.
| `filter` | `JsonArgument` | Narrows down the results based on [filters](#filtering).
| `sort` | `[String]` | [Sorts](#sorting) the results based on one or more fields and directions.

Example query and response:

```graphql
{
    terms {
        current_page
        data {
            id
            title
        }
    }
}
```

```json
{
    "terms": {
        "current_page": 1,
        "data": [
            { "id": "tags::one", "title": "Tag One" },
            { "id": "tags::two", "title": "Tag Two" }
        ]
    }
}
```

### Term {#term-query}

Used for querying a single taxonomy term.

```graphql
{
    term(id: "tags::one") {
        id
        title
    }
}
```

```json
{
    "term": {
        "id": "tags::one",
        "title": "Tag One"
    }
}
```

### Global Sets {#global-sets-query}

Used for querying multiple global sets.

Returns a list of [GlobalSetInterface](#global-set-interface) types.

| Argument | Type | Description |
|----------|------|-------------|
| `taxonomy` | `[String]` | Narrows down the results by terms in one or more taxonomies.
| `limit` | `Int` | The number of results to be shown per paginated page.
| `page` | `Int` | The paginated page to be shown. Defaults to `1`.
| `filter` | `JsonArgument` | Narrows down the results based on [filters](#filtering).
| `sort` | `[String]` | [Sorts](#sorting) the results based on one or more fields and directions.

Example query and response:

```graphql
{
    globalSets {
        title
        handle
        ... on GlobalSet_Social {
            twitter
        }
        ... on GlobalSet_Company {
            company_name
        }
    }
}
```

```json
{
    "globalSets": [
        { "handle": "social", "twitter": "@statamic" },
        { "handle": "company", "company_name": "Statamic" },
    ]
}
```

### Global Set {#global-set-query}

Used for querying a single global set.

```graphql
{
    globalSet(handle: "social") {
        title
        handle
        ... on GlobalSet_Social {
            twitter
        }
    }
}
```

```json
{
    "globalSet": {
        "title": "Social",
        "handle": "social",
        "twitter": "@statamic",
    }
}
```

### Navs {#navs-query}

Used for querying Navs.

```graphql
{
    navs {
        handle
        title
    }
}
```

```json
{
    "navs": [
        { "handle": "header_links", "title": "Header Links" },
        { "handle": "footer_links", "title": "Footer Links" },
    ]
}
```

### Nav {#nav-query}

Used for querying a single Nav.

```graphql
{
    nav(handle: "footer") {
        handle
        title
    }
}
```

```json
{
    "nav": {
        "handle": "footer",
        "title": "Footer Links"
    }
}
```

## Custom Queries

Here's an example of a basic query class. It has the name attribute which is the key the user needs to put in the request, any number of middleware, the type(s) that will be returned, any arguments, and how the data should be resolved.

```php
use Statamic\Facades\GraphQL;
use Statamic\GraphQL\Queries\Query;

class Products extends Query
{
    protected $attributes = [
        'name' => 'products',
    ];

    protected $middleware = [
        MyMiddleware::class,
    ];

    public function type(): Type
    {
        return GraphQL::paginate(GraphQL::type(ProductType::NAME));
    }

    public function args(): array
    {
        return [
            'limit' => GraphQL::int(),
        ];
    }

    public function resolve($root, $args)
    {
        return Product::paginate($args['limit']);
    }
}
```

```graphql
{
    products {
        name
        price
    }
}
```

You may add your own queries to Statamic's default schema.

You can add them to the config file, which makes sense for app specific queries:

```php
// config/statamic/graphql.php
'queries' => [
    MyCustomQuery::class
]
```

Or, you may use the `addQuery` method on the facade, which would be useful for addons.

```php
GraphQL::addQuery(MyCustomQuery::class);
```

## Types

- [EntryInterface](#entry-interface)
- [Collection](#collection-type)
- [CollectionStructure](#collection-structure-type)
- [CollectionTreeBranch](#collection-tree-branch-type)
- [NavTreeBranch](#nav-tree-branch-type)
- [PageInterface](#page-interface)
- [TermInterface](#term-interface)
- [AssetInterface](#asset-interface)
- [GlobalSetInterface](#global-set-interface)
- [Code](#code-type)

### EntryInterface {#entry-interface}

| Field | Type | Description |
|-------|------|-------------|
| `id` | `ID!` |
| `title` | `String!` |

Each `EntryInterface` will also have implementations for each collection/blueprint combination.

You will need to query the implementations using fragments in order to get blueprint-specific fields.

```graphql
{
    entries {
        id
        title
        ... on Entry_Blog_Post {
            intro
            content
        }
        ... on Entry_Blog_ArtDirected_Post {
            hero_image
            content
        }
    }
}
```

The fieldtypes will define their types. For instance, a text field will be a `String`, a [grid](#grid-fieldtype) field will expose a list of `GridItem` types.

### Collection {#collection-type}

| Field | Type | Description |
|-------|------|-------------|
| `handle` | `String!` |
| `title` | `String!` |
| `structure` | [`CollectionStructure`](#collection-structure-type) | If the collection is structured (e.g. a "pages" collection), you can use this to query its tree.

### CollectionStructure {#collection-structure-type}

| Field | Type | Description |
|-------|------|-------------|
| `handle` | `String!` |
| `title` | `String!` |
| `tree` | [[`CollectionTreeBranch`](#collection-tree-branch-type)] | A list of tree branches.

### CollectionTreeBranch {#collection-tree-branch-type}

Represents a branch within a structured collection's tree.

| Field | Type | Description |
|-------|------|-------------|
| `depth` | `Int!` | The nesting level of the current branch.
| `entry` (or `page`) | [`EntryInterface`](#entry-interface) | Contains the entry's fields.
| `children` | [[`CollectionTreeBranch`](#collection-tree-branch-type)] | A list of tree branches.

:::tip
It's not possible to perform recursive queries in GraphQL. If you want to retrieve multiple levels of child branches, take a look at a workaround in [recursive tree branches](#recursive-tree-branches) below.
:::

### NavTreeBranch {#nav-tree-branch-type}

Represents a branch within a nav's tree.

| Field | Type | Description |
|-------|------|-------------|
| `depth` | `Int!` | The nesting level of the current branch.
| `page` | [`PageInterface`](#page-interface) | Contains the page's fields.
| `children` | [[`NavTreeBranch`](#nav-tree-branch-type)] | A list of tree branches.

:::tip
It's not possible to perform recursive queries in GraphQL. If you want to retrieve multiple levels of child branches, take a look at a workaround in [recursive tree branches](#recursive-tree-branches) below.
:::


### PageInterface {#page-interface}

A "page" within a nav's tree.

| Field | Type | Description |
|-------|------|-------------|
| `id` | `ID!` | The ID of the page.
| `entry_id` | `ID` | The `entry` ID.
| `title` | `String` | For entry pages, it's the entry's `title` unless overridden on the branch. For basic pages, it's the `title`.
| `url` | `String` | For entry pages, it's the entry's `url`. For basic pages, it's the `url`. For text-only pages it'll be null.
| `permalink` | `String` | The absolute version of `url`.

If you want to query any fields that you've added to the nav's blueprint, you have 4 different options available to you that you can use as inline fragments.
You can use more than one at a time:

- `EntryInterface` for all entry pages.
- `NavEntryPage_{NavHandle}_{Collection}_{Blueprint}` for a specific entry/blueprint combination on entry pages.
- `NavBasicPage_{NavHandle}` for basic non-entry pages.
- `NavPage_{NavHandle}` for either basic or entry pages.

```graphql
page {
    title
    url
    ... on EntryInterface {
        # ...
    }
    ... on NavPage_HeaderLinks {
        # ...
    }
    ... on NavBasicPage_HeaderLinks {
        # ...
    }
    ... on NavEntryPage_HeaderLinks_Blog_ArtDirected {
        # ...
    }
}
```

### TermInterface {#term-interface}

| Field | Type | Description |
|-------|------|-------------|
| `id` | `ID!` |
| `title` | `String!` |
| `slug` | `String!` |

Each `TermInterface` will also have implementations for each taxonomy/blueprint combination.

You will need to query the implementations using fragments in order to get blueprint-specific fields.

```graphql
{
    terms {
        id
        title
        ... on Term_Tags_RegularTag {
            content
        }
        ... on Term_Tags_SpecialTag {
            how_special
            content
        }
    }
}
```

The fieldtypes will define their types. For instance, a text field will be a `String`, a [grid](#grid-fieldtype) field will expose a list of `GridItem` types.

### AssetInterface {#asset-interface}

| Field | Type | Description |
|-------|------|-------------|
| `path` | `String!` | The path to the asset.

Each `AssetInterface` will also have an implementation for each asset container's blueprint.

You will need to query the implementations using fragments in order to get blueprint-specific fields.

```graphql
{
    entries {
        path
        ... on Asset_Images {
            alt
        }
    }
}
```

The fieldtypes will define their types. For instance, a text field will be a `String`, a [grid](#grid-fieldtype) field will expose a list of `GridItem` types.

### GlobalSetInterface {#global-set-interface}

| Field | Type | Description |
|-------|------|-------------|
| `handle` | `String!` | The handle of the set.
| `title` | `String!` | The title of the set.

Each `GlobalSetInterface` will also have an implementation for each set's blueprint.

:::tip
While Statamic doesn't enforce a blueprint for globals (see [Blueprint is Optional](/globals#blueprints-are-optional)), it _is_ required within the GraphQL context. Fields that haven't been explicitly added to a blueprint will not be available.
:::

You will need to query the implementations using fragments in order to get blueprint-specific fields.

```graphql
{
    globalSets {
        handle
        ... on GlobalSet_Social {
            twitter
        }
    }
}
```

The fieldtypes will define their types. For instance, a text field will be a `String`, a [grid](#grid-fieldtype) field will expose a list of `GridItem` types.

### Code {#code-type}

| Field | Type | Description |
|-------|------|-------------|
| `code` | `String!` | The actual code value.
| `mode` | `String!` | The language "mode".

The [code fieldtype](/fieldtypes/code) will return this type when `mode_selectable` is enabled. Otherwise, it'll just be a string.

```graphql
{
    snippet {
        code
        mode
    }
}
```


## Filtering

You can filter the results of listing queries (like `entries`) using the `filter` argument. This argument accepts a JSON object containing different
[conditions](/conditions).

```graphql
{
    entries(filter: {
        title: { contains: "rad", ends_with: "!" }
    }) {
        data {
            title
        }
    }
}
```

```json
{
    "data": [
        { "title": "That was so rad!" },
        { "title": "I wish I was as cool as Daniel Radcliffe!" },
    ]
}
```

If you only need to do a simple "equals" condition, then you can use a string and omit the condition name, like the `rating` here:

```graphql
{
    entries(filter: {
        title: { contains: "rad" }
        rating: 5
    }) {
        # ...
    }
```

If you need to use the same condition on the same field more than once, you can use the array syntax:

```graphql
{
    entries(filter: {
        title: [
            { contains: "rad" },
            { contains: "awesome" },
        ]
    }) {
        # ...
    }
```

## Sorting

You can sort the results of listing queries (like `entries`) on one or multiple fields, in any direction.

```graphql
{
    entries(sort: "title") {
        # ...
    }
```

```graphql
{
    entries(sort: "title desc") {
        # ...
    }
```

```graphql
{
    entries(sort: ["price desc", "title asc"]) {
        # ...
    }
```

## Pagination

Some queries (like [entries](#entries-query)) will provide their results using pagination.

In a paginated response, you will find the actual items within a `data` key.

By default there will be `1000` per page. You can change this using a `limit` argument.
You can specify the current paginated page using the `page` argument.

```graphql
{
    entries(limit: 15, page: 2) {
        current_page
        has_more_pages
        data {
            # ...
        }
    }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `data` | [mixed] | A list of items on the current page. In an `entries` query, there will be `EntryInterface` types, etc.
| `total` | `Int!` | Number of total items selected by the query.
| `per_page` | `Int!` | Number of items returned per page.
| `current_page` | `Int!` | Current page of the cursor.
| `from` | `Int` | Number of the first item returned.
| `to` | `Int` | Number of the last item returned.
| `last_page` | `Int!` | The last page (number of pages).
| `has_more_pages` | `Boolean!` | Determines if cursor has more pages after the current page.


## Fieldtypes

### Replicator

Replicator fields require that you query each set using a separate fragment.

The fragments are named after your configured sets using StudlyCased field and set handles. e.g. `Set_{ReplicatorFieldName}_{SetHandle}`

```yaml
fields:
  -
    handle: content_blocks
    field:
      type: replicator
      sets:
        image:
          fields:
            -
              handle: image
              type: assets
              max_files: 1
        pull_quote:
          fields:
            -
              handle: quote
              field:
                type: textarea
            -
              handle: author
              field:
                type: text
```

```graphql
{
    content_blocks {
        ... on Set_ContentBlocks_Image {
            type
            image
        }
        ... on Set_ContentBlocks_PullQuote {
            type
            quote
            author
        }
    }
}
```

:::tip
If you have nested fields, include each parent's handle, (and grandparent's, great grandparent's etc), like so: `Set_TopLevelReplicator_NestedReplicator_DeeplyNestedReplicator_SetHandle`
:::

### Bard

Bard fields work the same as Replicator, except that you also have an additional `BardText` for the text fragment.

```graphql
{
    content_blocks {
        ... on BardText {
            type
            text
        }
        ... on Set_ContentBlocks_Image {
            type
            image
        }
        ... on Set_ContentBlocks_PullQuote {
            type
            quote
            author
        }
    }
}
```

### Grid

Grid fields can be queried with no extra requirements. You can just use the nested field handles.

```graphql
{
    cars {
        make
        model
    }
}
```

### Select, Radio, Checkboxes, and Button Group

These fieldtypes provide you with labels and values. You'll need to use a sub selection.

```graphql
my_select_field {
    value
    label
}
```

```json
"my_single_select_field": {
    "value": "potato",
    "label": "Potato"
}
```

The same syntax is used when multiple values are expected. e.g. a select field with multiple values enabled, or a checkboxes field. You'll just get a nested array returned.

```json
"my_multi_select_field": [
    {
        "value": "potato",
        "label": "Potato"
    },
    {
        "value": "tomato",
        "label": "Tomato",
    }
]
```

## Recursive Tree Branches

Often, when dealing with navs, you need to recursively output all the child branches. For example, when using the `nav` tag in Antlers, you might do something like this:

```
<ul>
{{ nav }}
    <li>
        <a href="{{ url }}">{{ title }}</a>
        {{ if children }}
            <ul>{{ *recursive children* }}</ul>
        {{ /if }}
    </li>
{{ /nav }}
</ul>
```

In GraphQL, it's not possible to perform recursive queries like that. You'll need to explicitly query each level:

```graphql
{
    nav(handle: "links") {
        tree {
            page {
                title
                url
            }
            children {
                page {
                    title
                    url
                }
                children {
                    page {
                        title
                        url
                    }
                }
            }
        }
    }
}
```

In this example, if you wanted anything more than `title` and `url`, you'd need to add them to each level.

This can quickly become tedious and is very repetitive, so here's a workaround using fragments.

If you wanted to add more fields, you only need to do it one spot - the `Fields` fragment. If you want to query more levels, you can just increase the nesting level of the `RecursiveChildren` fragment.

```graphql
{
    nav(handle: "links") {
        tree {
            ...Fields
            ...RecursiveChildren
        }
    }
}

fragment Fields on NavTreeBranch {
    depth
    page {
        title
        url
        # any other fields you want for each branch
    }
}

fragment RecursiveChildren on NavTreeBranch {
    children {
        ...Fields
        children {
            ...Fields
            children {
                ...Fields
                # just keep repeating this as deep as necessary
            }
        }
    }
}
```

Hat tip to Hash Interactive for their [blog post](https://hashinteractive.com/blog/graphql-recursive-query-with-fragments/) on this technique.

## Custom Fieldtypes

A fieldtype can define what GraphQL type will be used. By default, all fieldtypes will return strings.

```php
use GraphQL\Type\Definition\Type;

public function toGqlType()
{
    return GraphQL::string();
}
```

You're free to return an array with a more complicated structure in order to provide arguments, etc.

```php
use GraphQL\Type\Definition\Type;

public function toGqlType()
{
    return [
        'type' => GraphQL::string(),
        'args' => [
            //
        ]
    ];
}
```

If you need to register any types, the fieldtype can do that in the `addGqlTypes` method:

```php
public function addGqlTypes()
{
    // A class that extends Rebing\GraphQL\Support\Type
    $type = MyType::class; // or `new MyType;`

    GraphQL::addType($type);
}
```

## Laravel Package

Under the hood, Statamic uses the [rebing/graphql-laravel](https://github.com/rebing/graphql-laravel) package.

By default, the integration should feel seamless and you won't even know another package is being used. Statamic will
perform the following automatic configuration of this package:

- Setting up the `default` schema to Statamic's.
- Disabling the `/graphiql` route (since we have our own inside the Control Panel)

However, you're free to use this package on its own, as if you've installed it into a standalone Laravel application.

If Statamic detects that you've published the package's config file (located at `config/graphql.php`), it will assume you're trying to use it manually and will
avoid doing the automatic setup steps mentioned above.

If you'd like to use Statamic's GraphQL schema within the config file (maybe you want a different default, and want Statamic's one at `/graphql/statamic`)
you can use the `DefaultSchema` class.

```php
[
    'schemas' => [
        'statamic' => \Statamic\GraphQL\DefaultSchema::config()
    ]
]
```

## Authorization

By default, all queries are allowed by anyone. We plan to add native features in the future.

You can define custom authorization logic for any query by providing a closure to the static `auth` method.

```php
EntriesQuery::auth(function () {
    return true; // true authorizes, false denies.
});
```

## Custom Fields

You can add fields to certain types by using the `addField` method on the facade.

The method expects the [type](#types) name, the field name, and a closure that return a GraphQL field definition array.

For example, if you wanted to include a thumbnail from an asset, you could do that here. You can even have arguments. In this example, we'll expect the width of the thumbnail to be passed in.

```php
use GraphQL\Type\Definition\Type;
use Statamic\Facades\GraphQL;
use Statamic\Facades\Image;
use Statamic\Facades\URL;

GraphQL::addField('EntryInterface', 'thumbnail', function () {
    return [
        'type' => GraphQL::string(),
        'args' => [
            'width' => [
                'type' => GraphQL::int(),
            ]
        ],
        'resolve' => function ($entry, $args) {
            $asset = $entry->augmentedValue('image')->value();
            $url = Image::manipulate($asset)->width($args['width'])->build();
            return URL::makeAbsolute($url);
        }
    ];
});
```

```graphql
{
    entry(id: 1) {
        thumbnail(width: 100)
    }
}
```

```json

{
    "entry": {
        "thumbnail": "http://yoursite.com/img/asset/abc123?w=100"
    }
}
```

The closure you pass to the method should return a GraphQL field definition array.

You may add custom fields to the following types and any of their implementations:

- `EntryInterface`
- `PageInterface`
- `TermInterface`
- `AssetInterface`
- `GlobalSetInterface`

## Caching

GraphQL uses a basic whole-response cache by default. Each query/variables combination's response will be cached for an hour. You may customize the cache expiry in `config/statamic/graphql.php`.

```php
'cache' => [
    'expiry' => 60,
],
```

### Cache Invalidation

Cached responses are automatically invalidated when content is changed. Depending on your GraphQL usage and blueprint schema, you may also wish to ignore specific events when invalidating.

```php
'cache' => [
    'expiry' => 60,
    'ignored_events' => [
        \Statamic\Events\UserSaved::class,
        \Statamic\Events\UserDeleted::class,
    ],
],
```

### Disabling Caching

If you wish to disable caching altogether, set `cache` to `false`.

```php
'cache' => false,
```

## Custom Middleware

You may add custom middleware, which are identical to any other Laravel middleware class. They will be executed on all GraphQL requests (unless another middleware, e.g. caching, prevents it).

Use the `handle` method to perform some action, and pass the request on.

```php
use Closure;

class MyMiddleware
{
    public function handle($request, Closure $next)
    {
        // do something

        return $next($request);
    }
}
```

You may add your own middleware to Statamic's default schema.

You can add them to the config file, which makes sense for app specific middleware:

```php
// config/statamic/graphql.php
'middleware' => [
    MyMiddleware::class
]
```

Or, you may use the `addMiddleware` method on the facade, which would be useful for addons.

```php
GraphQL::addMiddleware(MyMiddleware::class);
```

## Troubleshooting

**"Cannot query field" error"**

If you see an error like `Cannot query field "entries" on type "Query"`, this likely means you haven't enabled that query. See [Enable GraphQL](#enable-graphql).
After enabling it, you may need to clear your cache as the request would probably have been cached.
