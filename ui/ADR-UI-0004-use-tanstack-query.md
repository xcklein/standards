---
status: accepted
date: 2026-05-16
tags: [ui, react, data-fetching, state, tanstack]
---
# Use TanStack Query

## Directive

All server data fetching must use TanStack Query's `useQuery` and `useMutation` hooks. Direct `fetch` calls inside components or `useEffect` data fetching are not permitted.

## Context and Problem Statement

UI applications must fetch, cache, and synchronise server state. Managing this manually with `useEffect` and `useState` leads to inconsistent loading and error states, stale data, redundant network requests, and complex cache invalidation logic. A dedicated server state library handles these concerns uniformly, leaving components responsible only for rendering.

## Decision Drivers

* Server state must be cached and shared across components without prop drilling
* Loading, error, and success states must be handled consistently across the application
* Stale data must be revalidated automatically in the background
* The library must integrate with `openapi-fetch` for type-safe queries

## Considered Options

* TanStack Query (React Query)
* Redux Toolkit Query
* Manual `useEffect` / `useState`

## Decision Outcome

Chosen option: "TanStack Query", because it provides a complete server state solution with caching, background revalidation, and fine-grained loading/error state management, and integrates cleanly with `openapi-fetch` typed clients.

### Examples

Wrap the application with `QueryClientProvider`:

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

export function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Router />
    </QueryClientProvider>
  );
}
```

A typed query using `openapi-fetch`:

```tsx
import { useQuery } from "@tanstack/react-query";
import { useFetch } from "@/hooks/use-fetch.ts";

export function useMe() {
  const { client } = useFetch({ getToken });

  return useQuery({
    queryKey: ["me"],
    queryFn: async () => {
      const { data, error } = await client.GET("/v1/me");
      if (error) throw error;
      return data;
    },
  });
}
```

Invalidate cache after a mutation:

```tsx
const queryClient = useQueryClient();

const mutation = useMutation({
  mutationFn: (patch) => client.PATCH("/v1/me", { body: patch }),
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ["me"] }),
});
```

### Consequences

* Good, because loading, error, and success states are handled uniformly across all queries
* Good, because automatic background revalidation keeps data fresh without manual polling
* Good, because cache deduplication prevents redundant requests for the same data
* Good, because `queryKey` arrays provide a predictable cache invalidation model
* Bad, because `QueryClientProvider` must wrap the component tree — adds a required setup step
* Bad, because cache key management requires discipline to avoid stale or over-broad invalidations

### Confirmation

All server data fetching in UI components must use TanStack Query's `useQuery` and `useMutation` hooks. Direct `fetch` calls inside components or `useEffect` data fetching are not permitted.

## Pros and Cons of the Options

### TanStack Query

* Good, because comprehensive server state solution — caching, deduplication, revalidation
* Good, because consistent loading/error state handling across the application
* Good, because integrates cleanly with any fetch client including `openapi-fetch`
* Bad, because adds a required provider and query client setup
* Bad, because cache key design requires care to avoid bugs

### Redux Toolkit Query

* Good, because deeply integrated with Redux for projects already using it
* Bad, because requires Redux — significant overhead if not already in use
* Bad, because more complex setup than TanStack Query

### Manual `useEffect` / `useState`

* Good, because no additional dependencies
* Bad, because loading, error, and caching logic must be reimplemented for every data fetch
* Bad, because prone to race conditions, stale closures, and inconsistent states
* Bad, because no cache deduplication — multiple components trigger duplicate requests

## More Information

* [TanStack Query documentation](https://tanstack.com/query)
* Related: [ADR-TS-0009 — Use openapi-fetch](../typescript/ADR-TS-0009-use-openapi-fetch.md)
