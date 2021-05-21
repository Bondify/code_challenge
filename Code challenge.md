# Problem Statement

Consider a (very) drunk person exiting a bar and trying to find their way home. Their house is on the same street as the bar, but, drunk as they are, they do not remember in which direction to walk. Out of despair, they decide to just randomly take steps to the left or the right until they arrive home. They have a slight limp, which makes their probability of stepping to the right (the positive direction) a bit more likely (**p_right=0.501**) than the left (**p_left=0.499**).

You may assume that the bar is at the origin, and that **the house is located at +20 steps from the bar**. Steps to the left and to the right always have the exact same size. Further, assume that the street has infinite length in the negative direction away from the house.

1. Give an estimate for the typical number of steps they will have to take until they make it to their house.
2. What happens if the "limp" is removed and **p_right = p_left**?  Why does this make solving this problem much harder?

We recommend you find an approximate solution by writing code that simulates this scenario. Please submit both your answer and the code you wrote.

## Readme

You can run the code yourself or directly run the graphs with the results explained below getting the data [here](https://github.com/Bondify/code_challenge/tree/main/data)

- results.csv: results of the first part of the experiment
- steps_story.json: steps story that feeds the graph in the "Curiosities" section
- results2.csv: results for the second part of the experiment

## Part 1

### Variables and parameters

- We are trying come up with a simulation that helps us understand how many steps our drunk friend has to take in order to get to his/her house. 
- We know the house is +20 steps to the right of the bar. Which means that, at some point, he/she has to accumulate 20 steps more in the right direction than in the left direction (if, for example, he/she starts taking 40 to the left and then takes 60 steps to the right, he/she would get home).
- We know this person is so drunk that they are walking randomly, but because of a slight limp they have, 50.1% of the times he/she will take a step in the right direction (and 49.9% of the times in the wrong one).
- When they take a step to the right we add 1 to the cumulative steps. When they take a step to the left we substract 1.  
- We know this person will keep walking until he/she hits the +20 in the right direction or to the infinite in the left direction. 
- In other words, the number of steps is the sample size our experiment has to have to get +20 given the probabilities we have right now.
- Then, explain why having p_right = p_left makes this problem much harder.

A possible experiment to simulate the situation desbribed above could be:

-  **Define the variables:** 
    - "cum_steps": The position the person is at a given point of the night (experiment). For each iteration of the experiment we start at cum_steps = 0 (the bar).
    - "n": The total number of steps he/she has taken at a given point. This will be the final sample size once our drunk friend gets home. For each iteration of the experiment we start at n = 0.
    - "N: number of times we run the experiments. We'll assume our friend has the bad habit of drinking too much and we'll simulate the situation 100 times.
    - "x": random number between 0 and 1 that will define in which direction our friend walks for each step he/she takes.

-  **Steps:**
    1. Initialize the variables: cum_steps = 0, n = 0, N = 0.
    2. Generate the random number "x".
        - If x < 0.501 => cum_steps = cum_steps + 1. This situation simulates our friend taking a step to the right (and getting one step closer home).
        - If x > 0.501 => cum_steps = cum_steps - 1. This situation simulates our friend taking a step to the left.
    3. Update n = n+1. The person has taken one more step. Our sample size increases.
    4. Repeat 2 and 3 until cum_steps = 20.
    5. When we hit cum_steps = 20, check "n". That's the number of steps the drunk person had to take in that specific iteration of the experiment.
    6. Update N = N+1. Run the experiment (repeat 1 to 5) until N = 100. Then we can see how the distribution of steps for our drunk friend was during in those 100 crazy nights.


```python
import pandas as pd
import numpy as np
from numpy import random, mean
import plotly.express as px
import plotly.graph_objects as go
```


```python
%%time

N = 100                                                 # number of times we'll run the experiment
n_list = []                                             # List with the sample size for N experiments
max_left = []                                           # list of the max position to the left for each N
steps_story = dict()                                    # complete position history for each N
p_right = 0.501                                         # probability of taking a step right

# Run the experiment N times
for i in range(N):
    # Reset the experiment
    n = 0                                                # number of steps
    cum_steps_list = []                                  # history of positions
    cum_steps = 0                                        # current position
    while cum_steps < 20:                                # Run until we get home (convergence)
        x = random.uniform(low=0, high=1)                # Generate a random number between 0 and 1
        n +=1                                            # We've taken one more step, update n
        if x < p_right:                                  # If x < 0.501, we took a step to the right
            cum_steps += 1                               # Our position now increases by 1 step.
        else:
            cum_steps -= 1                               # If x is not < 0.501 we took a step left and our position decreases.
        cum_steps_list = cum_steps_list + [cum_steps]    # Add our new position to our position history list
        
    n_list.append(n)                                     # We got home, add n for the i iteration to the list of "n"
    max_left.append(min(cum_steps_list))                 # Add the max left position for the i iteration to the list
    steps_story[i] = cum_steps_list                      # Add the step story for the i iteration to the list (dict)
```

    Wall time: 1min 53s
    

In the table below we can see the basic descriptive stats for the experiment. We notice:
- The mean number of steps our drunk friend has to take to get home is 4194. If we consider that each step is 0.5s this means it takes around 0.5x6299/60/24 = 1.45 DAYS to get home from the bar that is 20 steps away.
- The distribution has a wild variation though, with a standard deviation of **12928**. 
- The **mean - std < 0**. But, since we know it is imposible for a person to take a negative number of steps (not to be confused with the position which can be negative) this indicates us that the distribution is highly skewed to the right and that we can expect a few very big values of **n** to the right which have to be the responsible for such a big std.
- The minimun number of steps (n) that our drunk friend had to take to get home was **46**. This is remarkable and we will take a better look at that specific iteration of the experiment later.
- The maximun number of steps (n) that our drunk friend had to take to get gome was **103930**. We will also take a better look at this specific iteration as it can be an interesting example to *talk about convergence when p_right is slightly higher than p_left*.
- We can also see that the percentile 75 is 1800 which is very far from the max value of 103930, indicating the likelihood of outliers. Furthemore, the percentile 75 is less than half the mean and the median is 8 times lower than the mean. Again, this is another proof that the distribution is very skewed to the right.

Let's check the histogram and box plot and treat the outliers in the next few cells.


```python
# Describe the results
steps = pd.DataFrame(data = {'N':range(N), 'n':n_list, 'max_left':max_left})
steps.n.describe().apply(lambda x: round(x,0))
```




    count       100.0
    mean       4194.0
    std       12928.0
    min          46.0
    25%         238.0
    50%         513.0
    75%        1800.0
    max      103930.0
    Name: n, dtype: float64



To give us a better sense of the distribution we can also get the 5th and 95th percentile. We can see that the 95th percentile is 19533 steps, which is much larger than the 75th percentile we saw earlier. This is in line with the large std we saw in the previous step. 

For now, we can estimate the number steps our drunk friend will have to take in one of the following ways:
- There is a 95% chance that the number of steps our drunk friend has to take to get home is less than 19534. Not so insightful right?
- There is a 75% chance that the number of steps our drunk friend has to take to get home is less than 1800.
- There is a 95% change that the number of steps our drunk friend has to take to get home fall in between 91 and 39976.


```python
p95 = steps.n.quantile(0.95)
p2_5 = steps.n.quantile(0.025)
p97_5 = steps.n.quantile(0.975)

print('95th percentile = {}'.format(round(p95)))
print('95% conf. interval = ({},{})'.format(round(p2_5), round(p97_5)))
```

    95th percentile = 19534.0
    95% conf. interval = (91.0,39976.0)
    

From the histogram below we confirm our suspicion of a very long tail to the right. 
Also we can see that the bin with biggest number of repetitions is the one with lower values from 0 to 200 steps.
If we zoom in to that area of the plot (you can do this by "selecting" that area of the plot with your mouse) we see that the total number of experiments where the drunk person took less than 200 steps to get home is 21.


```python
import pandas as pd

fig_hist = px.histogram(steps, x='n', template='simple_white', nbins=1000, title = 'Histogram - Number of steps taken to get +20 for 100 interations of the experiment')
fig_hist.show()
```


<div>                            <div id="7cb131ef-5888-4972-bbb7-d3b4498c2390" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("7cb131ef-5888-4972-bbb7-d3b4498c2390")) {                    Plotly.newPlot(                        "7cb131ef-5888-4972-bbb7-d3b4498c2390",                        [{"alignmentgroup": "True", "bingroup": "x", "hovertemplate": "n=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "", "marker": {"color": "#1F77B4"}, "name": "", "nbinsx": 1000, "offsetgroup": "", "orientation": "v", "showlegend": false, "type": "histogram", "x": [1100, 3404, 714, 1274, 516, 108, 94, 452, 1804, 23598, 316, 466, 154, 1798, 432, 864, 3984, 96, 252, 362, 180, 224, 106, 936, 1494, 178, 574, 432, 1752, 103930, 358, 118, 490, 1696, 1446, 1934, 826, 448, 250, 502, 1260, 420, 38874, 352, 182, 282, 86, 282, 1352, 304, 152, 388, 198, 296, 88, 19320, 496, 356, 2672, 1314, 144, 232, 914, 240, 650, 164, 1280, 4348, 732, 120, 286, 152, 10320, 1992, 1166, 11154, 48356, 808, 2322, 202, 190, 376, 15490, 134, 40974, 3704, 514, 512, 17114, 11048, 1916, 46, 200, 3144, 676, 178, 2534, 2650, 3498, 632], "xaxis": "x", "yaxis": "y"}],                        {"barmode": "relative", "legend": {"tracegroupgap": 0}, "template": {"data": {"bar": [{"error_x": {"color": "rgb(36,36,36)"}, "error_y": {"color": "rgb(36,36,36)"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "baxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmapgl"}], "histogram": [{"marker": {"line": {"color": "white", "width": 0.6}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "rgb(237,237,237)"}, "line": {"color": "white"}}, "header": {"fill": {"color": "rgb(217,217,217)"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowhead": 0, "arrowwidth": 1}, "autotypenumbers": "strict", "coloraxis": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "colorscale": {"diverging": [[0.0, "rgb(103,0,31)"], [0.1, "rgb(178,24,43)"], [0.2, "rgb(214,96,77)"], [0.3, "rgb(244,165,130)"], [0.4, "rgb(253,219,199)"], [0.5, "rgb(247,247,247)"], [0.6, "rgb(209,229,240)"], [0.7, "rgb(146,197,222)"], [0.8, "rgb(67,147,195)"], [0.9, "rgb(33,102,172)"], [1.0, "rgb(5,48,97)"]], "sequential": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "sequentialminus": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]]}, "colorway": ["#1F77B4", "#FF7F0E", "#2CA02C", "#D62728", "#9467BD", "#8C564B", "#E377C2", "#7F7F7F", "#BCBD22", "#17BECF"], "font": {"color": "rgb(36,36,36)"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "radialaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}, "shapedefaults": {"fillcolor": "black", "line": {"width": 0}, "opacity": 0.3}, "ternary": {"aaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "baxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "caxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}}, "title": {"text": "Histogram - Number of steps taken to get +20 for 100 interations of the experiment"}, "xaxis": {"anchor": "y", "domain": [0.0, 1.0], "title": {"text": "n"}}, "yaxis": {"anchor": "x", "domain": [0.0, 1.0], "title": {"text": "count"}}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('7cb131ef-5888-4972-bbb7-d3b4498c2390');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


But 200 steps of variation might still be a lot. We can create our own bins every 100 steps and see how our data looks. 

Then, we can plot the histogram with these new bins and see what the most frequent group looks like.
I will do this process only for the iterations of the experiment that ended up having n<5000 so we can zoom in that area of the histogram.

Looking at this new version of the histogram (below) we can see that 13 iterations ended up between 100 and 200 steps and 5 times our drunk friend got home in less than 100 steps. Let's take a better look at those 5 experiments.


```python
# Discretize the data
# Create the bins and get the counts per bin
counts, bins = np.histogram(steps.n, bins=range(0, 5000, 100))

# Get the mid value for each bin
bins_labels = ['{}-{}'.format(bins[i],bins[i+1]) for i in range(len(bins)-1)]
bins = 0.5 * (bins[:-1] + bins[1:])

# Get the weighted average
weighted_avg = sum(counts*bins/sum(counts))

# Probability per bin (since we have N=100 it is just the count/100)
probs = counts/sum(counts)

# Plot the probabilities
fig = px.bar(x=bins_labels, y=counts, labels={'x':'steps (n)', 'y':'count'}, 
             template='simple_white', title = 'Histogram - Number of steps in 100 steps bins')
fig.show()
```


<div>                            <div id="2f63f7f6-abfa-4c5c-9ce0-2affc4e69484" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("2f63f7f6-abfa-4c5c-9ce0-2affc4e69484")) {                    Plotly.newPlot(                        "2f63f7f6-abfa-4c5c-9ce0-2affc4e69484",                        [{"alignmentgroup": "True", "hovertemplate": "steps (n)=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "", "marker": {"color": "#1F77B4"}, "name": "", "offsetgroup": "", "orientation": "v", "showlegend": false, "textposition": "auto", "type": "bar", "x": ["0-100", "100-200", "200-300", "300-400", "400-500", "500-600", "600-700", "700-800", "800-900", "900-1000", "1000-1100", "1100-1200", "1200-1300", "1300-1400", "1400-1500", "1500-1600", "1600-1700", "1700-1800", "1800-1900", "1900-2000", "2000-2100", "2100-2200", "2200-2300", "2300-2400", "2400-2500", "2500-2600", "2600-2700", "2700-2800", "2800-2900", "2900-3000", "3000-3100", "3100-3200", "3200-3300", "3300-3400", "3400-3500", "3500-3600", "3600-3700", "3700-3800", "3800-3900", "3900-4000", "4000-4100", "4100-4200", "4200-4300", "4300-4400", "4400-4500", "4500-4600", "4600-4700", "4700-4800", "4800-4900"], "xaxis": "x", "y": [5, 16, 11, 8, 8, 5, 3, 2, 3, 2, 0, 2, 3, 2, 2, 0, 1, 2, 1, 3, 0, 0, 0, 1, 0, 1, 2, 0, 0, 0, 0, 1, 0, 0, 2, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0], "yaxis": "y"}],                        {"barmode": "relative", "legend": {"tracegroupgap": 0}, "template": {"data": {"bar": [{"error_x": {"color": "rgb(36,36,36)"}, "error_y": {"color": "rgb(36,36,36)"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "baxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmapgl"}], "histogram": [{"marker": {"line": {"color": "white", "width": 0.6}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "rgb(237,237,237)"}, "line": {"color": "white"}}, "header": {"fill": {"color": "rgb(217,217,217)"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowhead": 0, "arrowwidth": 1}, "autotypenumbers": "strict", "coloraxis": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "colorscale": {"diverging": [[0.0, "rgb(103,0,31)"], [0.1, "rgb(178,24,43)"], [0.2, "rgb(214,96,77)"], [0.3, "rgb(244,165,130)"], [0.4, "rgb(253,219,199)"], [0.5, "rgb(247,247,247)"], [0.6, "rgb(209,229,240)"], [0.7, "rgb(146,197,222)"], [0.8, "rgb(67,147,195)"], [0.9, "rgb(33,102,172)"], [1.0, "rgb(5,48,97)"]], "sequential": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "sequentialminus": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]]}, "colorway": ["#1F77B4", "#FF7F0E", "#2CA02C", "#D62728", "#9467BD", "#8C564B", "#E377C2", "#7F7F7F", "#BCBD22", "#17BECF"], "font": {"color": "rgb(36,36,36)"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "radialaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}, "shapedefaults": {"fillcolor": "black", "line": {"width": 0}, "opacity": 0.3}, "ternary": {"aaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "baxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "caxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}}, "title": {"text": "Histogram - Number of steps in 100 steps bins"}, "xaxis": {"anchor": "y", "domain": [0.0, 1.0], "title": {"text": "steps (n)"}}, "yaxis": {"anchor": "x", "domain": [0.0, 1.0], "title": {"text": "count"}}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('2f63f7f6-abfa-4c5c-9ce0-2affc4e69484');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


We could try to see how this experiment would look removing the outliers. Below, we remove the outliers and plot two box plots, one with all the data and one with no outliers.

Already from the descriptive statistics we can see that the tail is now much shorter. The minimun value has remained the same (which means we didn't have any outliers in that direction) and the max has decreases significatly.

We can also check that in the box plots below.


```python
q3 = steps.n.quantile(.75)
q1 = steps.n.quantile(.25)

iqr = q3 - q1

upper = q3 + 1.5*iqr
lower = q1 - 1.5*iqr

no_outliers = steps.loc[
    (steps.n < upper) &
    (steps.n > lower)
]

no_outliers.n.describe().map(int)
```




    count      88
    mean      851
    std       933
    min        46
    25%       201
    50%       450
    75%      1263
    max      3984
    Name: n, dtype: int64




```python
fig1 = px.box(no_outliers, x='n', template='simple_white', height=300)
fig2 = px.box(steps, x='n', template='simple_white', height=300)

fig1.show()
fig2.show()
```


<div>                            <div id="f3613410-a673-4e3c-b1dd-289d333d8513" class="plotly-graph-div" style="height:300px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("f3613410-a673-4e3c-b1dd-289d333d8513")) {                    Plotly.newPlot(                        "f3613410-a673-4e3c-b1dd-289d333d8513",                        [{"alignmentgroup": "True", "hovertemplate": "n=%{x}<extra></extra>", "legendgroup": "", "marker": {"color": "#1F77B4"}, "name": "", "notched": false, "offsetgroup": "", "orientation": "h", "showlegend": false, "type": "box", "x": [1100, 3404, 714, 1274, 516, 108, 94, 452, 1804, 316, 466, 154, 1798, 432, 864, 3984, 96, 252, 362, 180, 224, 106, 936, 1494, 178, 574, 432, 1752, 358, 118, 490, 1696, 1446, 1934, 826, 448, 250, 502, 1260, 420, 352, 182, 282, 86, 282, 1352, 304, 152, 388, 198, 296, 88, 496, 356, 2672, 1314, 144, 232, 914, 240, 650, 164, 1280, 732, 120, 286, 152, 1992, 1166, 808, 2322, 202, 190, 376, 134, 3704, 514, 512, 1916, 46, 200, 3144, 676, 178, 2534, 2650, 3498, 632], "x0": " ", "xaxis": "x", "y0": " ", "yaxis": "y"}],                        {"boxmode": "group", "height": 300, "legend": {"tracegroupgap": 0}, "margin": {"t": 60}, "template": {"data": {"bar": [{"error_x": {"color": "rgb(36,36,36)"}, "error_y": {"color": "rgb(36,36,36)"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "baxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmapgl"}], "histogram": [{"marker": {"line": {"color": "white", "width": 0.6}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "rgb(237,237,237)"}, "line": {"color": "white"}}, "header": {"fill": {"color": "rgb(217,217,217)"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowhead": 0, "arrowwidth": 1}, "autotypenumbers": "strict", "coloraxis": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "colorscale": {"diverging": [[0.0, "rgb(103,0,31)"], [0.1, "rgb(178,24,43)"], [0.2, "rgb(214,96,77)"], [0.3, "rgb(244,165,130)"], [0.4, "rgb(253,219,199)"], [0.5, "rgb(247,247,247)"], [0.6, "rgb(209,229,240)"], [0.7, "rgb(146,197,222)"], [0.8, "rgb(67,147,195)"], [0.9, "rgb(33,102,172)"], [1.0, "rgb(5,48,97)"]], "sequential": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "sequentialminus": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]]}, "colorway": ["#1F77B4", "#FF7F0E", "#2CA02C", "#D62728", "#9467BD", "#8C564B", "#E377C2", "#7F7F7F", "#BCBD22", "#17BECF"], "font": {"color": "rgb(36,36,36)"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "radialaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}, "shapedefaults": {"fillcolor": "black", "line": {"width": 0}, "opacity": 0.3}, "ternary": {"aaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "baxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "caxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}}, "xaxis": {"anchor": "y", "domain": [0.0, 1.0], "title": {"text": "n"}}, "yaxis": {"anchor": "x", "domain": [0.0, 1.0]}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('f3613410-a673-4e3c-b1dd-289d333d8513');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>



<div>                            <div id="adce879f-93e5-4fc0-b786-62823441ee13" class="plotly-graph-div" style="height:300px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("adce879f-93e5-4fc0-b786-62823441ee13")) {                    Plotly.newPlot(                        "adce879f-93e5-4fc0-b786-62823441ee13",                        [{"alignmentgroup": "True", "hovertemplate": "n=%{x}<extra></extra>", "legendgroup": "", "marker": {"color": "#1F77B4"}, "name": "", "notched": false, "offsetgroup": "", "orientation": "h", "showlegend": false, "type": "box", "x": [1100, 3404, 714, 1274, 516, 108, 94, 452, 1804, 23598, 316, 466, 154, 1798, 432, 864, 3984, 96, 252, 362, 180, 224, 106, 936, 1494, 178, 574, 432, 1752, 103930, 358, 118, 490, 1696, 1446, 1934, 826, 448, 250, 502, 1260, 420, 38874, 352, 182, 282, 86, 282, 1352, 304, 152, 388, 198, 296, 88, 19320, 496, 356, 2672, 1314, 144, 232, 914, 240, 650, 164, 1280, 4348, 732, 120, 286, 152, 10320, 1992, 1166, 11154, 48356, 808, 2322, 202, 190, 376, 15490, 134, 40974, 3704, 514, 512, 17114, 11048, 1916, 46, 200, 3144, 676, 178, 2534, 2650, 3498, 632], "x0": " ", "xaxis": "x", "y0": " ", "yaxis": "y"}],                        {"boxmode": "group", "height": 300, "legend": {"tracegroupgap": 0}, "margin": {"t": 60}, "template": {"data": {"bar": [{"error_x": {"color": "rgb(36,36,36)"}, "error_y": {"color": "rgb(36,36,36)"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "baxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmapgl"}], "histogram": [{"marker": {"line": {"color": "white", "width": 0.6}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "rgb(237,237,237)"}, "line": {"color": "white"}}, "header": {"fill": {"color": "rgb(217,217,217)"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowhead": 0, "arrowwidth": 1}, "autotypenumbers": "strict", "coloraxis": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "colorscale": {"diverging": [[0.0, "rgb(103,0,31)"], [0.1, "rgb(178,24,43)"], [0.2, "rgb(214,96,77)"], [0.3, "rgb(244,165,130)"], [0.4, "rgb(253,219,199)"], [0.5, "rgb(247,247,247)"], [0.6, "rgb(209,229,240)"], [0.7, "rgb(146,197,222)"], [0.8, "rgb(67,147,195)"], [0.9, "rgb(33,102,172)"], [1.0, "rgb(5,48,97)"]], "sequential": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "sequentialminus": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]]}, "colorway": ["#1F77B4", "#FF7F0E", "#2CA02C", "#D62728", "#9467BD", "#8C564B", "#E377C2", "#7F7F7F", "#BCBD22", "#17BECF"], "font": {"color": "rgb(36,36,36)"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "radialaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}, "shapedefaults": {"fillcolor": "black", "line": {"width": 0}, "opacity": 0.3}, "ternary": {"aaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "baxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "caxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}}, "xaxis": {"anchor": "y", "domain": [0.0, 1.0], "title": {"text": "n"}}, "yaxis": {"anchor": "x", "domain": [0.0, 1.0]}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('adce879f-93e5-4fc0-b786-62823441ee13');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


### Curiosities

In the graph below we see the 5 iterations where our drunk friend got home in less than 100 steps.

The y axis is the number of steps taken while the x axis shows the position (home being +20).

All of these are pretty impressive but iteration 91 shows us our friend "converging" home after only 46 steps. This is stricking!

If we keep only iteration 91 in the graph (you can hide traces by clicking on the legend), it is even more stricking to see that from step #5 (the drunk was at +6 by then) to step #14 (when he/she gets home) the drunk took all the 9 steps in the right direction, ON A ROW! Furthermore, it was never in a negative position.

That is extremely unlikely. Actually, the probability of that happening is exactly = 0.501^9 = 0.0000627665361377109 which is a bit lower than 1/500.


```python
# Plot the samples where the drunk got home faster
fig = go.Figure()

fastest = steps.loc[steps.n<100]

for i in fastest.N:
    fig.add_trace(go.Scatter(
    #     mode = 'lines+markers',
        name = f'Iteration {i}',
        x = steps_story[i],
        y = list(range(len(steps_story[i])))
    ))

fig.update_layout(template='simple_white', title = 'Step history of the 6 shortest experiments')
fig.update_xaxes(title_text='Position', showgrid=True)
fig.update_yaxes(title_text='Steps', showgrid=True)
fig.show()
```


<div>                            <div id="24884023-91cf-401e-a7a3-9a3d188abf77" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("24884023-91cf-401e-a7a3-9a3d188abf77")) {                    Plotly.newPlot(                        "24884023-91cf-401e-a7a3-9a3d188abf77",                        [{"name": "Iteration 6", "type": "scatter", "x": [-1, -2, -1, -2, -3, -4, -5, -4, -3, -4, -3, -4, -3, -2, -1, 0, 1, 0, 1, 2, 1, 2, 3, 2, 1, 2, 3, 4, 5, 4, 3, 2, 3, 4, 3, 4, 5, 6, 5, 6, 5, 6, 7, 6, 7, 6, 7, 6, 5, 6, 7, 8, 9, 10, 9, 10, 9, 8, 7, 8, 9, 10, 11, 12, 13, 14, 13, 14, 15, 14, 15, 16, 17, 16, 17, 16, 15, 16, 17, 18, 17, 18, 19, 18, 19, 18, 19, 18, 19, 18, 17, 18, 19, 20], "y": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93]}, {"name": "Iteration 17", "type": "scatter", "x": [1, 2, 1, 0, 1, 2, 3, 4, 5, 4, 5, 6, 5, 4, 5, 6, 5, 6, 5, 4, 5, 4, 5, 6, 7, 8, 9, 10, 9, 10, 11, 10, 11, 10, 9, 10, 11, 10, 11, 10, 11, 10, 9, 10, 9, 10, 9, 8, 7, 6, 7, 8, 9, 10, 11, 10, 11, 12, 13, 14, 15, 14, 15, 16, 15, 16, 17, 16, 17, 18, 17, 16, 15, 16, 17, 16, 15, 14, 13, 12, 11, 12, 13, 14, 15, 14, 15, 16, 17, 16, 17, 18, 17, 18, 19, 20], "y": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95]}, {"name": "Iteration 46", "type": "scatter", "x": [1, 2, 1, 0, -1, -2, -3, -2, -3, -2, -1, -2, -1, 0, 1, 2, 1, 2, 1, 2, 3, 4, 3, 4, 5, 6, 5, 6, 5, 6, 5, 6, 7, 6, 5, 6, 5, 6, 7, 8, 9, 10, 9, 10, 11, 10, 11, 12, 11, 10, 11, 12, 13, 14, 13, 12, 11, 10, 9, 10, 9, 10, 11, 10, 11, 12, 13, 14, 13, 12, 13, 14, 15, 14, 15, 14, 15, 14, 15, 16, 17, 18, 17, 18, 19, 20], "y": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85]}, {"name": "Iteration 54", "type": "scatter", "x": [-1, -2, -1, 0, -1, -2, -1, -2, -3, -2, -1, -2, -3, -2, -3, -2, -1, 0, -1, 0, -1, 0, 1, 2, 1, 2, 3, 4, 3, 2, 3, 2, 3, 4, 3, 4, 3, 4, 3, 2, 3, 4, 5, 6, 7, 6, 5, 4, 3, 2, 1, 2, 3, 2, 3, 2, 3, 4, 5, 4, 5, 6, 7, 6, 7, 8, 9, 10, 11, 12, 11, 12, 13, 14, 15, 16, 17, 16, 17, 18, 19, 18, 17, 18, 17, 18, 19, 20], "y": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87]}, {"name": "Iteration 91", "type": "scatter", "x": [1, 2, 3, 4, 5, 4, 5, 6, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 13, 12, 13, 12, 13, 12, 13, 12, 11, 12, 13, 14, 13, 14, 15, 16, 15, 14, 15, 16, 17, 18, 19, 18, 19, 18, 19, 20], "y": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45]}],                        {"template": {"data": {"bar": [{"error_x": {"color": "rgb(36,36,36)"}, "error_y": {"color": "rgb(36,36,36)"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "baxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmapgl"}], "histogram": [{"marker": {"line": {"color": "white", "width": 0.6}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "rgb(237,237,237)"}, "line": {"color": "white"}}, "header": {"fill": {"color": "rgb(217,217,217)"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowhead": 0, "arrowwidth": 1}, "autotypenumbers": "strict", "coloraxis": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "colorscale": {"diverging": [[0.0, "rgb(103,0,31)"], [0.1, "rgb(178,24,43)"], [0.2, "rgb(214,96,77)"], [0.3, "rgb(244,165,130)"], [0.4, "rgb(253,219,199)"], [0.5, "rgb(247,247,247)"], [0.6, "rgb(209,229,240)"], [0.7, "rgb(146,197,222)"], [0.8, "rgb(67,147,195)"], [0.9, "rgb(33,102,172)"], [1.0, "rgb(5,48,97)"]], "sequential": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "sequentialminus": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]]}, "colorway": ["#1F77B4", "#FF7F0E", "#2CA02C", "#D62728", "#9467BD", "#8C564B", "#E377C2", "#7F7F7F", "#BCBD22", "#17BECF"], "font": {"color": "rgb(36,36,36)"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "radialaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}, "shapedefaults": {"fillcolor": "black", "line": {"width": 0}, "opacity": 0.3}, "ternary": {"aaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "baxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "caxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}}, "title": {"text": "Step history of the 6 shortest experiments"}, "xaxis": {"showgrid": true, "title": {"text": "Position"}}, "yaxis": {"showgrid": true, "title": {"text": "Steps"}}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('24884023-91cf-401e-a7a3-9a3d188abf77');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


We could now take a look at the iteration that took longest to converge and see how that looks.
To do so I filter the iteration with the biggest sample size and look for its steps story. In this case, it is iteration 29 out of the 100.

We can see that at some point the drunk guy was almost 500 steps to the left of the bar.
It took this person more than 100k steps to get home. If we consider each step takes them 0.5s that would be 0.5*103924/60/24 = 36 DAYS (!!) to get home.

No doubt this encourages us to control what we drink.

**Nonetheless, since this person has a limp that makes a bit more probable to walk to the right than
to the left, when leaving the experiment run infintively, it will converge to the right.** 

In other words, sooner or later, the drunk guy will get home. We just need to let the experiment run "indefintely".
That is why for this part of the experiment we didn't set a max for the steps left the drunk could take.

In the next part of the problem where p_right=p_left, the possibility of never converging really exists and we'll have to set a limit for the size each iteration can take. Trying to be as fair as we can, we can set the limit at the max size sample as the max size we found in part 1: 103924 steps.


```python
# Sample with the max steps left (more negative value)
max_left_n = steps.loc[steps.n == steps.n.max(), 'N'].values[0]

# Plot the experiment where the drunk took longer to get home
fig = go.Figure()

fig.add_trace(go.Scatter(
    name = 'Max steps left',
    x = steps_story[max_left_n],
    y = list(range(len(steps_story[max_left_n])))
))

fig.update_layout(template='simple_white', title = 'Step history for the longest iteration')
fig.update_xaxes(title_text='Position', showgrid=True)
fig.update_yaxes(title_text='Steps')
fig.show()
```



var gd = document.getElementById('291e8db7-4452-4a0f-b3cc-45f9a277cff7');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


## Part 2

In this part we have to answer the question:

> What happens if the "limp" is removed and p_right = p_left? Why does this make solving this problem much harder?

In the first case where p_right was slightly higher than p_left, we had one event (giving a step) and two possible outcomes. Since the positive outcome (step right) was slighty more probable than a negative outcome, we could rest assure that sooner or later, our drunk friend would get home. Every time we run the experiment, sooner or later, will converge to +20 steps (or to the positive direction more generally speaking).

If p_right = p_left this is no longer the case. Now, both outcomes have the exact same probability and it is equally probable for our friend to go left or right. In this case, each time we run the experiment, if we leave it running for big enough sample sizes, it should converge to 0 and our drunk friend would find him/her self exactly were everything started: at the bar.
 
There's really no guarantee our drunk friend will ever get home, even if in some iterations of the experiments we might get to the +20 position. This is similar to the case we saw earlier where our drunk friend almost got to -500 steps from the bar, even when the probability of going left was slight lower than the probability of going right. Even in that case, as we the experiment ran, it started converging to the right (just like the theory tells us it should).

Since we don't want to problem to run indefinitely we will stop each iteration when the sample size gets bigger than the max we found in part 1 (103924). It is still an arbitrary measure but it is just to ilustrate the point. 

Then, we can check how many times we hit that position. When we hit that position we will assign n = inf to ilustrate the non convergence.


```python
%%time

N = 100                                                 # number of times we'll run the experiment
n_list = []                                             # List with the sample size for N experiments
max_left = []                                           # list of the max position to the left for each N
steps_story = dict()                                    # complete position history for each N
p_right = 0.5                                           # probability of taking a step right

# Run the experiment N times
for i in range(N):
    # Reset the experiment
    n = 0                                                # number of steps
    cum_steps_list = []                                  # history of positions
    cum_steps = 0                                        # current position
    while (cum_steps < 20) & (n < 103924):               # Run until we get home (convergence) or n > n_max
        x = random.uniform(low=0, high=1)                # Generate a random number between 0 and 1
        n +=1                                            # We've taken one more step, update n
        if x < p_right:                                  # If x < 0.501, we took a step to the right
            cum_steps += 1                               # Our position now increases by 1 step.
        else:
            cum_steps -= 1                               # If x is not < 0.501 we took a step left and our position decreases.
        cum_steps_list = cum_steps_list + [cum_steps]    # Add our new position to our position history list
        
    n_list.append(n)                                     # We got home, add n for the i iteration to the list of "n"
    max_left.append(min(cum_steps_list))                 # Add the max left position for the i iteration to the list
    steps_story[i] = cum_steps_list                      # Add the step story for the i iteration to the list (dict)
```

    Wall time: 2min 18s
    


```python
# Describe the results
import math
steps2 = pd.DataFrame(data = {'N':range(N), 'n':n_list, 'max_left':max_left})
steps2.loc[steps2.n == 103924, 'n'] = math.inf
```




    count     100.0
    mean        inf
    std         NaN
    min        68.0
    25%       280.0
    50%       619.0
    75%      2578.0
    max         inf
    Name: n, dtype: float64



In this second experiment we can see that the experiment reached 103924 steps 5 times. Under the conditions we specified, we could interpret it as the experiment not reaching +20 in any stage of the process even when leaving it to run to infinity.

This makes sense, since there's no real reason to think it should ever get to the +20 position. If anything, we should bet it will stay at the bar for the most part if we let it run indefinetely.


```python
non_conv = len(steps2.loc[steps2.n == math.inf])
print("The experiment didn't converge {} times".format(non_conv))
```

    The experiment didn't converge 5 times
    