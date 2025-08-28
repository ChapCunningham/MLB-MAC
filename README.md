# MLB-MAC

We spent the Summer with the Brewster Whitecaps in the Cape Cod Baseball League, working as analytics interns. When preparing for the season, we wanted to create a system in which we could utilize past data to help us understand upcoming batter/pitcher matchups for each game of the season. This understanding would help us provide the coaching staff with suggestions for ideal matchups out of the bullpen, pitch-type optimization to a given hitter, and identifying our own hitters who have a good matchup against the opponents starting pitcher.

The goal was to have this program be extremely interpretable, meaning that it could be understood by a long-time coach who has been in dugouts for decades without a full understanding of advanced analytics. At the same time, we wanted the process and programming to be at the level at which a research and development team member could appreciate.

The math behind MAC can be difficult to explain to those unfamiliar with the methods used, but the concept and, most importantly, the output is extremely easy to understand and interpret.

We were fortunate to be given the opportunity to present MAC at Saber Seminar on August 24th, 2025, below is a more in-depth summary of the presentation.

The video of our presentation can be found here.

First Iteration and Euclidean Distance
The first step of this project was to define similar pitches. There are a number of distance testing methods that can be used to find similar points of data, but we ultimately landed on Euclidean.

We went back and forth on what features to include, what features to leave out, and came to the conclusion that it’s vital to the program to intake only variables that the batter perceives. When it comes down to it, we’re very concerned with exclusively what the batter is able to see when he’s in the box. The way that we landed on the specific features that were included was by our own intuition, but also player feedback.

If you’ve ever been in a dugout and talked to a hitter after an at-bat, you’ve likely heard one of the following:

“This guy’s got a super funky slot”

“His heater is riding a ton”

“Man, that slider is really spinning”

Encapsulating all of these thoughts that hitters mention in the dugout was important information for us to use when creating the list of scanning features.

Scanning Features: [Velocity, Induced Vertical Break, Horizontal Break, Spin Rate, Release Height, Release Side, *Arm Angle*]
*Arm Angle is included in the MLB iteration of MAC, but not the NCAA iteration*

The process of deeming pitches as similar or not similar was tricky and took a lot of trial and error. To understand how the math behind finding out if a pitch is similar or not, here is the euclidean distance equation used to find that distance value from one pitch to another.

![picture_1](https://github.com/user-attachments/assets/95cf530f-2841-4ba4-987b-e34c26690a7b)


*Variables are standardized in order to make everything equal when scanning for similarity* — For each variable, x1 would be the pitcher’s pitch and x2 would be a pitch the hitter has seen
This distance equation was applied to each pitch that the chosen hitters have seen in the database, then if it fell within our distance threshold, it was added to the similar pitches data frame.

We landed on a distance threshold of 0.6. From a long process of the aforementioned trial and error, as well as an understanding for the database of pitches and the average distance between two random pitches in the dataset, 0.6 was the “sweet-spot”.

Ultimately, selecting the distance threshold is a tradeoff between two things:

Having a significant sample of similar pitches that the hitters have seen
Ensuring that the threshold is small enough to only capture pitches that are truly similar to those of the input pitcher
To further understand and visualize how 0.6 was chosen, here’s the distribution of distances for a random sample of MLB pitches. These distances below are found by choosing two pitches at random and noting their euclidean distance using the scanning features mentioned above.

![pic_2](https://github.com/user-attachments/assets/5b789ad8-eea6-4440-a72b-6c7e5cffe491)


Random Sample of 3000 pitches from MLB season (2024 & 2025)
We confirm that the distance threshold of 0.6 is truly gathering only the most similar pitches to the input pitcher.

Side Note: We heavily considered implementing a dynamic threshold, one that would adjust itself according to the sample size that it returned for that given Pitcher vs. Lineup input, however, for now we’ve decided against it. We feel as though altering the threshold in exchange for a more consistent sample would create a less consistent result in terms of output. When the threshold grows too large, naturally the pitches that are returned as similar become less and less similar.

![pic_3](https://github.com/user-attachments/assets/a35b9651-a83a-442e-b0b4-00600bd42454)


Example of three different pitches against a “Target Pitch” — taken from 8/24/25 presentation at Saber Seminar
The above graphic does a good job at explaining exactly how this hard threshold works.

We have a Target Pitch, which looks to be a LHP Fastball. Database Pitch A returns a distance of 0.45, meaning that it is confirmed as similar. Database Pitch B, while relatively similar in each of the scanning feature variables, is just outside of the threshold because of the aggregate minute disparities from itself to the target pitch. Database Pitch C is clearly not a Fastball, it’s likely a ChangeUp, so for this particular fastball, Pitch C is deemed not similar.

It’s important to know that this process runs for every pitch the input pitcher has thrown.

![pic_4](https://github.com/user-attachments/assets/d4adae5a-3061-4bc3-acd6-2a435ff2941c)
First example of MAC (pitcher and hitter name excluded for data sharing regulation purposes)


Above is the very first version of MAC. It was an incredibly simple table of hitter vs. pitcher matchups that simply scanned for similar pitches (relative to the inputted pitcher) that the inputted lineup of hitters have faced.

So, the filtering process for similarity was complete, but we were still struggling to fully encapsulate a true matchup report because we had not accounted for usage and individual pitch types.

This is where clustering comes into play.

Second Iteration and Clustering
Just like distance testing, there are plenty of ways to cluster data. We played around with a number of different methods and did extensive research, both baseball and non-baseball related to land on the exact method that worked best for the program’s needs.

We stumbled upon a FanGraphs article from 2021. Written by Cam Rogers, now with the Los Angels Angels in R&D, the article aimed to quantify bullpen matchups with the use of clustering. This article was wildly valuable to us, giving us an idea as to how to apply the logic of a Gaussian Mixture Model into our clustering process.

The article can be found here. We strongly recommend you reading it.

In order to cluster as accurately as possible, we define a set of clustering features. These features were chosen with the intention to effectively separate pitches into individual groups.

Clustering Features: [Velocity, Induced Vertical Break, Horizontal Break, Spin Rate, Spin Axis]
The amount of clusters is chosen using similar logic to that of Rogers. We use GMM to cluster the pitches and find the optimal amount of clusters by using BIC scores to identify the Elbow Point.

In plain terms, we iterate through n amount of clusters until the optimal amount of clusters is found. This process is often referred to as the elbow method.

The GMM is trained and the amount of clusters is chosen based on the input pitcher’s data alone, that way, we keep the target goal of the program at the forefront of the process.

Once the optimal amount of clusters is chosen and the corresponding model is selected, the similar pitches are iterated through to correctly place them in a cluster.

![pic_5](https://github.com/user-attachments/assets/70f00516-3349-4af0-8dde-7d6d21f8b0c7)
Example Clustering: NCAA RHP — 5-pitch-mix


Above is an example of a clustered college pitcher. We’ve specifically used this as the example because of the visual overlap between clusters. The mix of green and blue pitches that are clearly breaking balls are split in that way due to deviations in spin and velocity.

This movement plot encapsulates the difficulty in clustering pitches. Pitchers, specifically those that are still at the collegiate level, can struggle to recreate pitch shapes consistently. We can see that holding true with the lone cluster 4 pitch (purple) that is visually grouped near cluster 1 (orange). That lone purple plot is indeed a changeup, but with noticeably more vertical break than the rest of cluster 4.

Due to the inconsistencies with not only the pitch shapes themselves at the college level, but also the data collection, we decided on a slightly altered approach to the next step of the MAC process.

For NCAA MAC, a pitcher’s clusters are grouped into just three separate possible groups: Fastball, Breaking Ball, and Offspeed.

![pic_6](https://github.com/user-attachments/assets/3ea55010-e434-49d7-9c0d-1bc810b7e1fd)

Second Iteration of MAC (Usage: NCAA Data / Names Blurred for Data Sharing Regulation Purposes)
Above is the example output of the second iteration of MAC. Now we have some sort of tangible usage for the matchup output. This visual was run using a local Dash app through Python. It was interactive where the user could hover over each of the plots and see the same stats displayed in the table, but split for each pitch type.

The colored plots: red, blue, green, and black represent the hitter’s performance against fastballs, breaking balls, offspeed pitches, and their overall results, respectively.

We knew that this kind of visual plot, where the hitters serve as the x-axis of the visual and the y-axis shows their overall performance in the form on their run value per 100 pitches (RV/100) was exactly the kind of simplistic report we were looking for.

The output for college data was (and still is) only broken up into the three different pitch groupings. For MLB data, we’re able to create more specific reports that are broken down into more granular detail.

The way that this grouping is done is by a majority voting system. Say for instance that Cluster A has 100 pitches in it. If 85 of those pitches are tagged (TaggedPitchType from TrackMan for NCAA or pitch_type from Baseball Savant CSV for MLB) as “Fastball” and 15 of them are tagged as “Sinker”, that group will be considered as the “Fastball” cluster.

With TaggedPitchType, we run the risk of using data that has been mistagged, but the use of it for this purpose has yet to cause any problems. For the most part, NCAA TrackMan taggers do a fine job of tagging most of the pitches correctly (80-grade TrackMan taggers, we thank you).

![pic_7](https://github.com/user-attachments/assets/bb6e0a9f-62a5-493f-8d60-dfd56740fe2e)

Here is an example of what the process would look like for the same NCAA RHP from earlier, color coded to show what the majority-vote determined each of their pitches as.

The blending of the Four-Seam and Two-Seam fastballs, as well as the Curveball and Slider make it so that the most optimal way to scout this pitcher is to group the pitches with a wider net. We’ve found that this logic holds to be true for the majority of college pitchers.

At the MLB level, the final product of a pitcher’s grouped arsenal would look something like this:

![pic_8](https://github.com/user-attachments/assets/0cfa6c65-b0a8-46b8-8b3e-335c34426e64)
Walker Buehler (and similar) Pitches using MAC’s Logic — Visual from mac-mlb.streamlit.app


Above is an example of Walker Buehler vs. a random sample of inputted hitters in the MAC application. The movement plot you see includes only pitches that the lineup of hitters have seen. Meaning that these pitches above might not include a single pitch of Buehler’s. It’s extremely unlikely (at the MLB level) to have this happen, but it is possible in the case that every selected hitter has never once faced the selected pitcher.

![pic_9](https://github.com/user-attachments/assets/5fa872b5-44c5-487d-b184-66b9e43f3fcd)
Walker Buehler 2025 Movement Plot — Baseball Savant

As you can see, MAC did a very good job at finding pitches that are truly similar to Buehler’s arsenal. Things that are slightly incorrect with this example is the Slider and Sweeper being aggregated together to just one pitch, rather than 2. This is a problem that sorts itself out with an increase in sample.

The MAC App
Now that the math and programming portion of the process is out of the way, onto the fun part. We’ll now walk you through the features that the MAC application allows users to utilize.

Overview

![pic_10](https://github.com/user-attachments/assets/73201042-eff0-4afe-b5da-ce470acfe683)

Upon launch, you’ll see a prompt for a pitcher, list of hitters, as well as the available bullpen arms for that day. Adding a bullpen aspect to this allows coaches and team-workers to understand the most ideal matchups for the pitcher prior to game time, or live in-game.

![pic_11](https://github.com/user-attachments/assets/db7c8cd8-c815-41b1-94a0-6bf50e79306e)

Once the “Run Complete MAC Analysis” button is clicked, the program begins searching and applying all of the logic explained in this article. This run-time is much quicker than we initially expected and allows for the program to be used and lineups/pitchers to be loaded without skipping a beat.

The above Hitter Summary shows a similar visual to the one from earlier (MAC Iteration #2).

![pic_12](https://github.com/user-attachments/assets/18db58fe-7fe2-4875-8cd4-049efd270b03)

The user can hover over (on computers or laptops) or click (on phones or tablets) on each of the plot points to get a more descriptive report of the hitter for that specific pitch type. Remember that the larger black dot represents the overall matchup.

The overall matchup dot is weighted to scale for the pitcher’s usage percentages for each pitch type. That way, we exclude sample bias for a hitter that has seen, for example, a large percentage of breaking balls from the similar pitch sample for this given input pitcher.

![pic_13](https://github.com/user-attachments/assets/ecd1256d-2031-49b5-8814-b3fa9eb6d926)

![pic_14](https://github.com/user-attachments/assets/a281ee5e-3f5d-4b1f-a6a0-3f23778eae25)

Just underneath the hitter summary visual graphic, the user will see a pair of tables. One for the overall summary of each hitter versus the pitcher and one that is split for each individual pitch type.

These tables can be exported as a CSV, for any pre-game prep purpose.

![pic_15](https://github.com/user-attachments/assets/4d0f0753-7c5f-4a9f-ae7b-031bdeef22d7)

Aaron Judge vs. Jack Flaherty (and similar pitches)
Following the tables is a 3x3 grid of heat maps that display whiff rate, hard hit rate, and wOBA versus each pitch group (Fastball, Breaking, Offspeed).

![pic_16](https://github.com/user-attachments/assets/a52a7e5d-1548-4846-8cf4-2363204b3b8d)

Finally, MAC provides a table of every possible matchup for that game. Each available pitcher is shown on the left and each column represents a hitter’s performance in RV/100.

This is useful for bullpen planning, as coaches can see what pockets of the lineup MAC thinks a pitcher is more or less favorable in.

MAC’s Usage
For our individual purpose, MAC was created to live in a dugout. This summer, we were able to utilize the interface in-game as a living scouting report for our opponents.

![pic_17](https://github.com/user-attachments/assets/cd9ff11e-54c9-4723-a92e-3d415e49d69c)
MAC in-game usage


MAC was also used as a scouting report tool. It was the ultimate filter when scouting an opponent. We were able to use it for our pitchers, obviously, but MAC can be reversed engineered to help maximize lineup construction for hitters.

Knowing how your hitters perform against a specific pitch shape, and not just the generalized usage of platoon splits, can pay off. With MAC, it was possible to get a closer, and more accurate look into projecting future matchups.

While MAC is not a model, and consequently not predictive, it does do a great job of preparing a coaching staff, players, etc. for what they can expect to see on game-day.

MLB MAC is available to the public at mac-mlb.streamlit.app. Feel free to check it out and play around with the tools available!

We’ve found it to be an interesting test to the program to have it up and running with a live MLB game to see how MAC’s suggestions align (or differ) with that of an MLB manager’s decision making
