# Rift Recap

This is our final project for the course DSC 106 at UCSD. Our website provides detailed visualizations from your recent League of Legends games and allows you to analyze your gameplay for potential patterns. We also provide feedback on your gameplay to help you identify your strenghts and weaknesses, hopefully leading to improvement.

At the bottom of this ReadMe are our design choices, such as the visualization techniques we chose and the story we were trying to tell with our project.

## First steps

To see your data, simply enter the region you're in, your game name (i.e. the part before the # in your Riot ID), and your tagline (the part after the #). If you don't have a Riot ID or don't wish to see your own stats, you can click the button provided to analyze the most recent ranked games for Faker, a popular professional player.

## The data you get


### Recap
The first visualization you'll see is an overall recap of your stats over the course of the past 30 games--such as your maximum kills, deaths, and assists, as well as your most picked items and runes. This information can give additional context to the information that follows--for example, you may see that you don't often score as many kills as you do assists, or you might often choose a build that doesn't do well early game.


### Bar Graph

The second visualization you'll see is a bar graph showing you how much you've played each champion in your last 30 games. We chose this visualization technique because it makes it easy for players to see how much time they've devoted to certain champions.

### Player Gold over Time Line Graph

The next visualization is an interactive line graph that showcases your gold over time for the champions you've played. For those unfamiliar with the terminology, gold is essentially a measure of strength in League--the more gold you have, generally, the stronger you are. By looking at the trajectories of the lines on the graph, you can see how well you perform at certain points with certain champions--for example, someone may see that they perform very poorly on Trundle until the 20 minute mark, at which their gold earnings start to increase. These lines can also be toggled, allowing for you to better identify the trends between specific champions.

### Best and Worst champions

After this, you'll be presented with your best and worst champions at various stages in the game--early game (laning phase), late game, and overall. This allows people who may not have understood the trends in the line graph to still see their strong and weak points.

### Opponent Gold over Time Line Graph

You'll also see a line graph for your opponents' gold over time, in the same way that your own data is presented. This allows you to see what champions you tend to struggle most against, and when those matchups are especially bad.

### Best and Worst Matchups

Once again, you'll be provided with a summary of your best and worst matchups (from the data in the line graph above). From this, you can see what your best and worst matchups are at various stages of the game.

### How to Improve

Finally, you'll see some tips on how to beat your three worst matchups. Specifically, you'll be able to see which champion in your pool does the best against these champions, and you can also see the point in the game at which you have the greatest advantage over these champions. This can help you build strategies moving forward on how to deal with these matchups.

## Design Choices

For our website, we chose a light gradient texture that would be easy on the eyes. It also allows the reader to better see the line graphs and bar graphs. We also decided to make our website scrollable, since this makes it easy to move through each section and analyze them one at a time.

We decided to use mutlicolored bar and line graphs so that our viewers could easily see the difference between the bars and lines and be able to tell them apart. We also added animation so that the visualizations and the following summaries would be more eye-catching.

We structured our website in accordance with the and-but-therefore structure. In essence:

You are presented with an overall recap and the champions you play most, **AND**
<br>
You can see how well you do on the various champions in your pool, **BUT**
<br>
The following champions depicted in the line graph are your worst counters, **THEREFORE**
<br>
You should take note of the following personalized tips in order to improve.




