# Contribution 1: Normalize instructor names

**Contribution Number:** 1
**Student:** Comfort Ohajunwa
**Issue:** https://github.com/mcgill-courses/mcgill.courses/issues/776 
**Status:** Phase 2 Complete

---

## Why I Chose This Issue

I chose this issue because it appears straightforward, but useful for students using the website. I also thought it could be an interesting challenging considering I'm less familiar with the coding languages. I hope to familiarize myself more with TypeScript and Rust, while hopefully making a meaningful contribution.

---

## Understanding the Issue

### Problem Description

Links for instructor pages are URL encoded. However, the repo maintainers would like to have clean, readable slugs for the instructor pages. 

### Expected Behavior

The instructor's name should be normalized (e.g., https://mcgill.courses/instructor/alida-souce)

### Current Behavior

The instructor's name is encoded (e.g., https://mcgill.courses/instructor/Alida%20Souc%C3%A9)

### Affected Components

The url for the instructor page is being constructed in the following places:

- client/src/components/course-terms.tsx (lines 120, 144)
- client/src/components/course-search-bar.tsx (lines 130, 178)
- client/src/components/course-review.tsx (line 394)
- client/src/components/course-averages.tsx (lines 28)

In each of these places, the instructor's name is encoded using `encodeURIComponent`.

---

## Reproduction Process

### Environment Setup

Setting up my local development environment was relatively straightforward. The only challenge I faced was running commands with `pnpm` because I was setting up the project on an exFAT drive. 

Error Message: `The "D:" drive is exFAT, which does not support symlinks. This will cause installation to fail. You can set the node-linker to "hoisted" to avoid this issue.`

This was resolved by running the relevant commands with additional flags:

- `pnpm install --config.node-linker=hoisted`
- `pnpm --config.verify-deps-before-run=false -r run dev`

### Steps to Reproduce

1. Spin up Docker container hosting MongoDB database (`docker compose up --no-recreate -d`)
2. Spawn Rust backend server (`cargo run -- --initialize --db-name=mcgill-courses`
3. Spawn React frontend (`pnpm -r run dev`)
4. Navigate to an instructor's page

### Reproduction Evidence

- **Branch link:** https://github.com/cohajunwa/mcgill.courses/tree/fix-issue-776
- **My findings:** During reproduction, I found the same problem identified in the issue. I also noticed that in some cases, there are several instructors with separate pages under different spellings. For instance Benoît Côté and Benoit Cote have separate pages storing slightly different data, despite being the same person (Benoît Côté likely being the correct spelling). This problem was noted in a separate [issue](https://github.com/mcgill-courses/mcgill.courses/issues/634).
---

## Solution Approach

### Analysis

The root cause is that the links to the instructor pages are URL encoded. When instructor information needs to be retrieved, the name is decoded back to its "raw form" and used as a key for the database.

### Proposed Solution

Add a slug field to the Instructor model that contains the normalized, "slugified" version of an instructor's display name.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The instructor pages needs to use human-readable slugs for the instructor name.

**Match:**

For course pages, the course ID is slugified (e.g., https://mcgill.courses/course/acct-695). There is a function that converts the retrieved course ID to a URL parameter (`client/src/lib/utils.ts:61-62): "COMP202" -> "comp-202". This function is called everytime the URL is being constructed. Then, to retrieve course data based on the course ID, the reverse operation is performed.

However, this approach will not be effective for normalizing instructor names given that names may have accents, apostrophes, hyphens, non-Latin scripts, etc. Creating a function that takes a slug and converts it to the original instructor's name to retrieve data would have to address various edge cases. For instance, the function would have to somehow know that `benoit-cote` maps to `Benoît Côté`. Instead, it would be more efficient to store a slug field for each instructor document in the database. 

**Plan:**

1. Create a function in `client/src/lib.utils.ts` called `instructorNameToUrlParam` that converts an instructor's name (e.g., Alida Souce) to a slug (e.g., alida-souce)
2. Add a new field in the Instructor model called `slug` (`crates/model/src/instructor.rs`, `client/src/lib/types.ts`)
3. Update database seeding logic so that instructors are added with a slug field (`crates/db/src/db.rs`)
4. Make server-side changes
   - Create a `db.find_instructor_by_slug` function (`src/instructors.rs`)
   - Update `get_instructor` function in `src/instructors.rs` to use `find_instructor_by_slug`
   - Update integration tests in `src/server.rs`
5. Make client-side changes
   - Update `client/src/lib/api.ts` so getInstructor takes a slug instead of a name.
   - Update the link sites (`client/src/components`) to use `instructor.slug` instead of `encodeUriComponent(name)`
   - Rename the route param from :name to :slug in `client/src/app.tsx`
   - Drop the `decodeURIComponent` used in `client/src/pages/instructor.tsx`
   - Update tests found in `client/src/pages/instructor.tests.tsx`
6. Write a one-time backfill migration script. Currently, the instructor documents in production does not contain a slug field. So, we would need to perform re-seeding to ensure that those entries contain the new field.

**Implement:** https://github.com/cohajunwa/mcgill.courses/tree/fix-issue-776

**Review:** The project does not have contribution guidelines. I will simply ensure my changes aligns with existing conventions.

**Evaluate:** Run automated tests.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
