# 2025 Tech Debt Roadmap

Owner: Brady Berryessa

## Q1

- A-team
    - Fix the timezone database download errors
        - We get errors when we deploy and start up Raven frequently these days trying to download a timezone database. If we can define where it comes from we should just point it at our own S3 bucket.
        - Medium priorty
        - 1 day
    - Refactor [Hotels.Booking](http://Hotels.Booking) and Hotels.Search
        
        
- Bookables
    - Upgrade to latest Elixir and OTP version for Assembler Storm and CAPI
    - Remove Old Wishlist Code
    - Guide Search Query Refactor
    - Encrypt CC bin
    - Travelpass Web Test coverage
    - Hotel Checkout page phone formatter
- Data
    - Project Horizon (https://linear.app/travelpass/project/project-horizon-214f136d483a/overview)
- Nitecrawler
    - Optimize hotel page JS
    - Need to optimize fetchers for load speed
- Platform
    
    Update versions of elixir and erlang in all repos
    
- Trips
    - Codebase cleanup
        
        
    - Squash n+1’s
    - Graph Audit
    - Migrate to Remix
    - Design System Audit
- Mobile
    - Completing applying design system to app
        1. Beginning of the year we started a redesign and we left it 60% completed. We are missing some screens to remove all the old code, apply the new components, and the restyle design system.
        2. It is a lot of work
        3. Yes, this one would be the most beneficial to address sooner
        4. Couple of months between testing and development
    - Removing unused components/screens/modals
        1. We have a bunch of old components/screens/stuff laying around that we should get rid of as they are cumbersome, can lead to using the incorrect components, and drives me nuts.
        2. It’s kind of painful because we need to manually check each file and see if it is used somewhere making sure that we don’t break anything. Pretty tedious.
        3. It can definitelly wait, nothing that is hindering performance or usability of the app. Just clutter.
        4. Couple of weeks between development and testing
    - Adding E2E tests for each feature
        1. Missing important flow tests to make the app more robust
        2. We are working on it starting with guides, but we should go back and add flows to all major features (search hotel/esperiences, book hotel/experiences, login, activity feed, etc)
        3. Not crucial to be implemeted right now, but definitely should be part of our roadmap for 2025 to have all old & new features with E2E testing
        4. Probably one week of focused effort or 2/3 weeks if diluted with other work

## Q2

- A-team
    - Replace outdated FE billing system
- Bookables
    
    Import Reviews from Expedia and Priceline to replace TPReviews
    
    Revisit hotel cards on TP Web
    
- Nitecrawler
    - Rework hotel favoriting API front-end architecture
- Trips
    - Oban Pro
    - Image Service
    - TypeScript Type Definitions
    - React Component Optimization
    

## Q3

- A-team
    - Fix provider_mappings map on rate search and rate request
- Bookables
    
    Consider Vervotech content as part of the assembler flow
    
- Trips
    - Move away from soft deletions
    - Decoupling trip creation + bookings
    - Expand Playwright Coverage
    - A11y Improvements

## Q4

- Trips
    - Find alternative places provider
    - Audit FE Dependencies
    - Map Component Refactor

## Not prioritized for 2025

- Bookables
    
    Raven CMS tech debt
    
    Rewrite Assembler into storm
    
- Data
    - Revisit development environments. Devs are spending extra time on this and we think it could be improved with trainings and documentation. Maybe we need to improve the process itself too.
    - We should rethink the shared layer. Should we start creating department specific schemas?
    - Deploy issues - sequence of files being deployed can be an issue but it’s more just annoying. Devs know how to work around it.
    - Lambda functions - we were planning to move them into the beast repo but with the changes in architecture coming (kafka), we probably want to wait to see if we even need lambdas anymore. So this is low priority and can be addressed probably Q3 2025.
- Mobile
    - Simplify auth
        1. We have an overly complicated auth state machine that we want to simplify and use the firebase functions directly, which will fix a nagging bug of an expired token that doesn’t refresh
        2. Pretty painful as it is an intricated piece of code and it will require careful testing to make sure nothing is broken
        3. The sooner the better I would say as it would resolve the token expiration issue which causes the app to show an error (not crash) sometimes
        4. Couple of weeks between development and testing
    - Organize better the hooks in the project
        1. We have repeating hooks that could be aggregated and reused. Three levels of hooks: project, feature, screen. We want to organize them in that way.
        2. Kinda painful as we need to check all the hooks in the project and see what we need to do.
        3. DEVX beneficial, but no change in the app itself
        4. Couple of weeks

# List of tech debt projects

## RavenCMS

- Still using material ui
- Give UX attention to tenant and winning logic UI. And/Or possibly pull tenant and winning logic out of RavenCMS and into a Phoenix app so backend devs can easily implement UI for their changes to the tenant schema there.

## Lucency

- Replace outdated FE billing system
    - This was created in a rush many years ago. It’s been hacked up to perform fine for what we need today.
    - Maybe this is low priority now that Chad is gone and we’re probably not trying to sell Lucency services again?
- Add automatic table partitioning to the FE billing so we don’t have to do it manually once a year
    - One of the hacks we had to make to the frontend billing system was to partition billing tables to make the process run faster. We just generated a bunch partitions by hand.

## IVR

- Token Auth (or any auth) implementation? [context here](https://travelpassgroup.slack.com/archives/C034P83U4J1/p1727191328719459)

## User Auth Service

- Upgrade to latest Elixir and OTP version

## CoverGenius

- Improved frontend validation
    
    Frontend apps implementing CG could be improved by restricting the offer if the customer is from a state that is not covered (NY, HI).
    

## Cable App (Sales and Support)

- Moving to Remix
    
    Cable does not need the SEO benefits of using Remix. Still, should Cable move to Remix to stay consistent with TP and NC? 
    
    Estimate: 1-2 weeks.
    
- Fix sales attribution issues
    
    Make booking and sales attribution in a single request https://linear.app/travelpass/issue/CAB-1376/adding-agentbookingrequest-payload-as-optional-to-createhotelbooking
    
    This is an ongoing problem and we have a dedicated Slack channel to deal with the bookings missing sales attribution info [#cable-missing-ids](https://travelpassgroup.slack.com/archives/C05QV15RQH4)
    
    ~~Estimate: 3-5 days.~~
    
- Remove the trip ID workaround (async booking)
    
    Cable uses Trip IDs as a workaround to check for a possible booking success after a network problem or a timeout. This approach is a misuse of the Trips and was put in check several times but so far we’re not able to find something reliable to use instead. 
    
- Improve client-side date validation
    
    The current validation around dates is messy and inconsistent. We tried to improve it a few times and stepped into issues related to the agent’s timezone versus the customer’s timezone. Cable makes +2k requests with invalid dates to Raven per week.
    
    Estimate: ~3-5 days.
    
- Refactor booking form components
    
    Currently, all form inputs in the booking page are wrapped by another component to allow using `react-hook-form` within the components of the Design system. It would be preferable to have components from the Design system ready to use `react-hook-form` [register](https://react-hook-form.com/docs/useform/register).
    
- Use ephemeral environments for a better CI/CD flow
    
    (…)
    
- Test Coverage
    
    Expand Playwright and unit tests for key pages/components to catch bugs early and improve reliability.
    

## Cable BE

- Upgrade to the latest Elixir and OTP version
    - Upgrade TZData lib

## Merchanting

- Fix the API interface
    - The interface requires too much knowledge of internals (having to pass processor-specific transaction data back to `merchanting` for settle/refund). I think the whole thing needs the attention of someone who’s not being rushed to add a new feature, because it hasn’t received that kind of TLC in years
    - The default behavior for errors is to charge the card, this is unexpected

## Content

- Upgrade to latest Elixir and OTP version

## Trips BE Architecture

- Find alternative place data provider
    
    If Google finds us caching responses and asks us to stop, we could get hit with a pretty huge google bill. It would be great to reduce our dependency on Google for Places. We should consider finding another provider, if possible. This could be a very complex problem, weeks to a month for discovery and implementation
    
- Codebase cleanup
    
    We have a lot of functionality defined at the api layer. We should invest some time to move logic out of this layer and into the trips app, where they can be better tested and reduce bloat in the API layer.
    
    Estimate ~5 days
    
- Squash n+1’s
    
    We have some n+1's in our setup, and we should move more logic into dataloader rather than one-off queries.
    
    Estimate: 3 days
    
- Graph Audit
    
    We have a bunch of stuff that we've deprecated, but haven't removed. It would be great to tidy up the graph schema and codebase a bit.
    
    Estimated time: 1-3 days
    
- Move away from soft deletions
    
    Soft deletion is a pain, we don't use the archived records, and it bloats the database. It would be awesome to move away from archival and use hard deletions instead.
    
    Estimate: 2-3 days
    
- Decoupling trip creation + bookings
    
    Cable uses a flow that creates a trip during the booking flow. They use these trips to help prevent ghost bookings. Supposedly, asynchronous bookings will remove the need for this, but we’ll need to pull it out with care.
    
    Estimate: 1 day if async bookings fixes use case, ??? if not.
    
- Setup Oban Pro for Raven
    
    We have jobs we don't have a ton of insight into. It would be great to wire this up so that we have better support for our jobs.
    
    Estimate- 2-3 days
    
- Create image service
    
    Right now, we expose our GCP api keys when you inspect network traffic. They are restricted to our specific [travelpass.com](http://travelpass.com/) referrer, but definitely not ideal. We probably want some sort of image service that helps keep these keys opaque
    
    Estimate: 1-2 weeks
    

## Bookables BE Tech Debt

- Remove Wishlist Code
    
    Wishlists were replaced by collections. We need to remove this from Raven this should have all been taken out from all of the graph consumers already. Just annoying to have a ton of unused code in our codebase. 1-2 days
    
- Encrypt CC Bin
    
    We save the entire Bin of a Credit card as part of bookings. With this and also saving the last 4 also saved on bookings we have most of the users credit card. We should consider encrypting the bin or look into what things we can do to be pci compliant. A couple days to a week depending on findings from pci [https://blog.pcisecuritystandards.org/8-digit-bins-and-pci-dss-what-you-need-to-know](https://blog.pcisecuritystandards.org/8-digit-bins-and-pci-dss-what-you-need-to-know)
    
- Guide Search Query Refactor
    
    Refactoring guide search graph queries as currently they are bloated with several args and query params that might be separated to different queries altogether. I personally feel that it may be wise to separate the guide search resolvers into its own resolver file to reduce clutter there as the rest of the guide logic is there as well. Refactoring the guide_search/1 function in guides.ex. There is some functionality in it that needs to be removed like the original keyword search functionality. The entire guide search function potentially be split out into clauses or different functions based on the needs of a specific query. The way it is now feels a bit like having to taste test a several coarse meal each time I look at updating something within guide search. The queries file itself might also need a look into to see if we could consolidate or remove some functionality thats not longer used or inefficient.Refactoring these things will make it easier to follow, update, and by removing old code and splitting things out might improve speed a bit as there will be less clauses to match on.
    
    Estimate: 2-3 weeks
    

## Travelpass Web App

- Migrate to Remix for SEO & Performance
    
    Current routing isn’t optimized for SEO or speed. Moving high-priority routes to Remix will enhance SEO, page load times, and user experience.
    
    For guides, we anticipate a 1 week lift
    
- TypeScript Type Definitions
    
    Strengthen type definitions across props, state, and API responses to reduce bugs and improve maintainability, focusing on complex components first
    
    2-3 day lift for dasbhoard/profile/guides
    
- Guides Map Component Refactor
    
    The current map implementation could be simplified and improved. Refactoring it to streamline the code and optimize performance will make it easier to maintain and customize in the future.
    
    3-5 days worth of work
    
- Audit imports/dependencies
    
    Remove outdated or unnecessary packages to reduce bundle size and simplify maintenance. 
    
    Estimate: 1-2 days
    
- A11y Improvements
    
    Address accessibility gaps (e.g., missing aria-* attributes) to ensure full user inclusivity and compliance.
    
- Test Coverage
    
    Expand Playwright tests for key flows (e.g., creating and interacting with guides) to catch bugs early and improve reliability.
    
    Estimates vary depending on scope
    
- Hotel Cards
    
    We currently have a large number of different hotel cards all pretty much displaying the same information. We should consider consolidating what we can so this can be more managable. Est about a week. https://linear.app/travelpass/issue/BOOK-2829/revisit-the-different-type-of-cards
    
- Hotel Checkout page phone formatter
    
    There are cleanup things around the formatter that should be fixed. There is a bandaid on it now but it is not extensible into international phone numbers. 1-2 days
    

## Nitecrawler Web App

- Need to optimize fetchers for load speed
    
    Currently our hotel and search pages primary queries have to wait until JS loads on the browser for the API to call to actually begin. We should move this onto the loader to have this data displayed as quickly as possible.
    
- Rework hotel favoriting API front-end architecture
    
    Our hotel favoriting system is reliant only on fetchers for any kind of revalidation after a mutation occurs. We should move this logic to loaders and fetchers for cleaner code and off loading a lot of that logic of the client and onto the server
    
- Optimize hotel page JS
    
    Looks like we’re getting errors in google lighthouse tied to our JS taking too long to run and blocking the main thread which leads to a reduced google quality score
    
    ![image.png](2025%20Tech%20Debt%20Roadmap%20134fc0e07c9980979c66fdaf04dd9ec7/image.png)
    

## Travelpass Mobile App

- Completing app redesign
    1. Beginning of the year we started a redesign and we left it 60% completed. We are missing some screens to remove all the old code, apply the new components, and the restyle design system.
    2. It is a lot of work
    3. Yes, this one would be the most beneficial to address sooner
    4. Couple of months between testing and development
- Simplify auth
    1. We have an overly complicated auth state machine that we want to simplify and use the firebase functions directly, which will fix a nagging bug of an expired token that doesn’t refresh
    2. Pretty painful as it is an intricated piece of code and it will require careful testing to make sure nothing is broken
    3. The sooner the better I would say as it would resolve the token expiration issue which causes the app to show an error (not crash) sometimes
    4. Couple of weeks between development and testing
- Removing unused components/screens/modals
    1. We have a bunch of old components/screens/stuff laying around that we should get rid of as they are cumbersome, can lead to using the incorrect components, and drives me nuts.
    2. It’s kind of painful because we need to manually check each file and see if it is used somewhere making sure that we don’t break anything. Pretty tedious.
    3. It can definitelly wait, nothing that is hindering performance or usability of the app. Just clutter.
    4. Couple of weeks between development and testing
- Adding E2E tests for each feature
    1. Missing important flow tests to make the app more robust
    2. We are working on it starting with guides, but we should go back and add flows to all major features (search hotel/esperiences, book hotel/experiences, login, activity feed, etc)
    3. Not crucial to be implemeted right now, but definitely should be part of our roadmap for 2025 to have all old & new features with E2E testing
    4. Probably one week of focused effort or 2/3 weeks if diluted with other work
- Organize better the hooks in the project
    1. We have repeating hooks that could be aggregated and reused. Three levels of hooks: project, feature, screen. We want to organize them in that way.
    2. Kinda painful as we need to check all the hooks in the project and see what we need to do.
    3. DEVX beneficial, but no change in the app itself
    4. Couple of weeks

## Provider Implementation

- None, we plan tech debt work as part of every quarter

## Hotels app

- Refactor [Hotels.Booking](http://Hotels.Booking) and Hotels.Search
    - These modules have become a mess of functions due to many contributors, but no owners.
    - Lower priority
    - Might take 2 - 3 days
- Fix provider_mappings map on rate search and rate request
    - It commits the following sins:
        - It isn’t typed in specs nor schema, but has a nested and necessary structure for Raven to work. To figure out the right structure you need to read many functions in many files to piece it together.
        - It changes shape mid-process when doing room matching.
        - Its structure slightly changes depending on if you’re running a search or a rate request
    - Lower priority, workarounds have been in place for a couple of years
    - 2-3 days

## General Raven

- Fix the timezone database download errors
    - We get errors when we deploy and start up Raven frequently these days trying to download a timezone database. If we can define where it comes from we should just point it at our own S3 bucket.
    - Medium priorty
    - 1 day
- Upgrade to latest Elixir and OTP version
    
    
- Fix broken dependency tree
    - Many individual apps inside of Raven can not compile because of poor dependency hygiene. This also increases the time time it takes to compile the app.
    - We may want to consider changing CI to run tests for each app individually
    - High priority
    - 3 - 4 days
    
- Fix flakey tests that worryingly don’t cause CI to fail
    - There are a number of flakey tests that fail from mocking Elixir stdlib modules. They show up as red in the tests, but don’t cause CI to fail. When I fixed one a few months ago it turned out the test it belonged to never worked.
    - After doing this I think we should present to the chapter what the problem is and how to avoid it
    - High priority
    - 3 - 4 days
- 

## Assembler / Importer

- Consider Vervotech content as part of assembler flow.
    1. We moved to using the new Vervotech endpoints last year which improved Vervotechs curated content. We do not consider this content currently in our assembler process. 
    2. It is stable currently but Priceline was confused why we were not using VVT content.
    3. It could wait for a bit but should be considered next year.
    4. A couple weeks
- Rewrite Assembler into Storm
    1. This is to rewrite the assembler service into the storm app. The assembler was our first elixir service and could use some updates. Also most people who wrote this are gone now so a lot of the context has been lost.
    2. This is more of assembler feels like a black box at times with lack of context
    3. It could wait and is probably lower priority than the above to consider VVT content as part of assembling.
    4. 1 - 1.5 months
- Upgrade to latest Elixir and OTP version
- Import Reviews from Expedia and Priceline to replace TPReviews
    - Vervotech does not get review data reliably so they suggested that we integrate with our provider apis for that information. High priority. 1 - 1.5 months.

## Logging POC

- Snowflake
- AWS
- Splunk

High priority tech debt:

- Project Horizon (https://linear.app/travelpass/project/project-horizon-214f136d483a/overview): This project is all about improving the efficiency of our data pipelines.
    - We have some circular dependencies at the shared layer. This makes it so we cannot change the lag times. We need to figure out where this is happening and a better path forward.
    - Revisit how we put data from unprocessed to processed. Can we use the same processed tables for multiple unprocessed tables? We should make it the standard. Nice if this is done by end of Q1 2025 to help support kafka architecture.

Medium priority tech debt:

- Revisit development environments. Devs are spending extra time on this and we think it could be improved with trainings and documentation. Maybe we need to improve the process itself too.
- We should rethink the shared layer. Should we start creating department specific schemas?

Low priority tech debt: 

- Deploy issues - sequence of files being deployed can be an issue but it’s more just annoying. Devs know how to work around it.
- Lambda functions - we were planning to move them into the beast repo but with the changes in architecture coming (kafka), we probably want to wait to see if we even need lambdas anymore. So this is low priority and can be addressed probably Q3 2025.