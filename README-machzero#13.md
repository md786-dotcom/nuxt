## Why this Repo

We have chosen this repo as we have previous experience in building webapps (through supabase) and websites through WordPress as a freelancer. So, this repo was close to heart. Previous experience in React.js made Nuxt preferable to us. Honestly, we don't have much experience with Vue but the repo is mostly in TS which i am familiar with, having built a couple of projects on it.

## Why this issue

Again this issue is close to us as its mostly focused on convenience and in the age of AI - convenience is the name of the game, and our team member's claudewp project was also based on convenience. (Claude to WordPress with native elementor edit compatibility. There was also not much activity in the issue itself (comments, assignment, Merge request etc) so this was preferable. Admittedly, triage was pending label was added by the bot, but the maintainer himself removed it. There are no approved/confirmed labels anywhere in the PRs - open or closed.

One might be thinking why there is a need for public API when prefetch hook is already there. Need is there because the built in prefetch only knows about Nuxt's objects/syntax, but many apps load with other tools/data - for example a custom query or a TanStack query. A public API like this would also act as a bridge between scheduler (which contains - throttle/deduplication and cancellation) and request.

## The issue itself

Nuxt has a prefetch scheduler. wet uses it to load pages and data before the user needs them. it does three tasks:

Throttling (Limit on how many requests run at the same time)
Deduplication (Removes/Fails the duplicate requests)
Cancellation (Stops requests that are no longer necessary)

Custom requests are put through the scheduler. so it might cause race conditiion - bugs which only happens when there are simultaneous requests. 

TanStack is a collection of software libraries and tools built for web development. internal, not part of public contract which can be changed without a break changing notice. 

## Working

<NuxtLink> detects intent - means signals that the user is likely to click the link soon. for example - hover. 
Default: wentersection Observer (link becomes visible)..

wet fires the link:prefetch hook
nuxtApp.hooks.callHook('link:prefetch', normalizedPath)

Listeners schedule work on the shared scheduler
Multiple plugins/composables listen and call schedule() with a task.

The scheduler runs the tasks with priority, concurrency limits, deduplication, and cancellation.

Nuxt's Prefetch by default:

route - page + layout + JS chunks
payload (JSON data)
island - isolated components that can be embedded in a page without shipping full JS to the client.
hint - destination tags

## Planning

- usePrefetchScheduler(return type - schedule | promote) and prefetchGroup(url) [PR uses to fill group field] to be made public and imported (automatic with a manual fallback) on every file.
1. For this, our approach go through 3 layers:
(a) remove the @internal tag - which results in breaking changes to be officially documented and notice to be given. 
(b) adding them in public export chain (series of files, each file gives a function to the next file and the last file is the entry point) [used because file paths can always change but the entry point stays the same] for proper import statement with #app. public export chain goes through multiple layers - internal (holds the code), composables [preload.ts - re-exports the functions]  [index.ts - adds the function to the export list]  (re-exports the two functions so that a function is available from a new file without a copy of code - one module export carries onto other modules)and finally the app/index.ts
(c) Nuxt has an auto import system called as unimport library that reads from a set of presets, the PR will add the two function names to the existing preload preset.

## Relevant Codebase

1.⁠ ⁠packages/nuxt/src/app/internal/prefetch-scheduler.ts - Prefetch Scheduler
2.⁠ ⁠packages/nuxt/src/app/internal/prefetch-util.ts - Group Helper
3.⁠ ⁠packages/nuxt/src/app/internal/prefetch.ts - Scheduler Core and Types

##  API Surface (Everything that a user can see and use)

Narrow PR; Easy to add, hard to remove - best practices

scheduler has 4 parts:

schedule(task) - public: #36655 directly asks for this. Only way to add work to the queue.
promote(group) - public: only changes the order of doing work. Users should be able to highlight parts that are important.
reset() - scheduler already calls it with 'scope: navigation'
resume() - Nuxt already does this automatically. 

## Alternative Approach

Alternative was a new high-level composable (wraps the scheduler and fills in the fields for us for comfortability) that hides the task object with a URL, callback part. But this was not chosen as it would require a lot of changes in existing codebase, and would take control away from users with fixed defaults and no visibility of task fields. 

## Testing

typecheck, syntax testing with linting tools. 

main focus will be on e2e tests: giving a custom prefetch query and checking if the 3 scheduler functions happens or not.






