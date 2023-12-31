# API design, contracts, agreements and best practices

## Common

-   Use camelCase for body params and response formats and kebab-case for path and query params
    -   Valid url: https://api.snmt.com/api/v1/project-activities/1?include_count=true
-   All list endpoints should be written as HTTP POST not GET.
    -   **Example:** instead of `GET /users` we're going to have `POST /users/search`
    -   **Why:** browser URL length limit is 2048 characters and because of filters (array of uuid v4 can exceed this limit). We're not going to wait for this moment and make all our list endpoints as POST by default.

## Pagination

-   Format:

```ts
export interface WithPaginationParams {
    limit?: number;
    offset?: number;
}
```

-   Every list endpoint should have pagination.
    -   **Exception:** There can be exceptions when endpoint returning an entity which count never gonna be more than 100 - in that case endpoint shouldn't have pagination.
-   By default endpoint should have max `limit` value being 100.
    -   **Exception:**
-   All list endpoints with pagination should return response in the following format:

```ts
export interface PaginatedResponse<Entity> {
    data: Entity[];
    metadata: {
        // Count is optional because by default it's not sent.
        // Consumer should use a include_count param to get count
        count?: number;
        limit: number;
        offset: number;
    };
}
```

-   All list endpoints by default don't return `metadata.count`. There's special parameter `{include_count: true}` to request count.
    -   **Why:** For optimization purposes. It's not cheap to calculate count and there're cases when requesting count is not needed:
        -   **Case 1:** Frontend optimization to request count only when user visits page for the first time. If user visited page 1 and then visited page 2 - there's no need to request count for page 2 because it's not changed.
            -   Note: don't forget to request count if filter or search params are changed
        -   **Case 2:** Service A wants to get users by list of ids from Service B - Service A doesn't need count for that case.

## Sorting

Since our list endpoints are POST instead of GET our sort format is:

-   Format:

```ts
interface WithSortParams<SortableColumns> {
    sort?: Array<{
        field: SortableColumns;
        order: SORT_OPTION;
    }>;
}

enum SORT_OPTION {
    ASC = 'asc',
    DESC = 'desc',
}
```

## Filters

-   Format:

```ts
interface WithFiltersParam<Fields extends string> {
    filters: FilterParam<Fields>[];
}

interface WithFiltersOrParam<Fields extends string> {
    filtersOr: FilterParam<Fields>[];
}

interface FilterParam<Fields extends string> {
    field: Fields;
    type: FilterType;
    operator: FilterOperator;
    value: FilterParamValue | FilterParamValue[];
}

type FilterType = 'text' | 'number' | 'date';
type FilterParamValue = string | number | boolean | null;

type FilterOperator =
    | ArrayFilterOperators
    | TextFilterOperators
    | NumberFilterOperators
    | DateFilterOperators;
type TextFilterOperators = 'equals';
type NumberFilterOperators =
    | 'equals'
    | 'lessThan'
    | 'greaterThan'
    | 'lessThanOrEqual'
    | 'greaterThanOrEqual';
type DateFilterOperators = 'equals' | 'lessThanEqual' | 'overlaps' | 'contains';
type ArrayFilterOperators = 'in';
```

## Search

-   Format:

```ts
interface WithSearchParam<Fields extends string> {
    search?: SearchParam<Fields>;
}

interface SearchParam<Fields extends string> {
    term: string;
    fields: Fields[];
}
```

-   Field `search.fields` responsible for listing fields that backend should search by.
-   **IMPORTANT:** Every list endpoint must support `search.fields` even if backend implementation searches only by 1 field today, tomorrow it can be more.
-   **Example:** client sends `POST /users/search` with body `{ search: { term: 'daniel', fields: ['fullName', 'email'] } }` which means backend should return all users where 'daniel' is used in email or full name

## Optional fields

TODO
