# Coronavirus tweets

You will scan all geotagged tweets sent in 2020 to monitor for the spread of the coronavirus on social media.

<!--
**Who should complete this assignment:**
only students who did not take cs46 with me
-->

**Due date:** 
Sunday, 5 March

This homework will require lots of computation time.
I recommend that you have your code working by 12 Feb to ensure that you will have enough time to execute the code.

**Learning Objectives:**

1. work with large scale datasets
1. work with multilingual text
1. use the MapReduce divide-and-conquer paradigm to create parallel code

## Background

Approximately 500 million tweets are sent everyday.
Of those tweets, about 2% are *geotagged*.
That is, the user's device includes location information about where the tweets were sent from.
The lambda server's `/data/Twitter dataset` folder contains all geotagged tweets that were sent in 2020.
In total, there are about 1.1 billion tweets in this dataset.

The tweets are stored as follows.
The tweets for each day are stored in a zip file `geoTwitterYY-MM-DD.zip`,
and inside this zip file are 24 text files, one for each hour of the day.
Each text file contains a single tweet per line in JSON format.
JSON is a popular format for storing data that is closely related to python dictionaries.

Vim is able to open compressed zip files,
and I encourage you to use vim to explore the dataset.

You will follow the MapReduce procedure to analyze these tweets.
This is a 3 step procedure summarized in the following image:

<img src=mapreduce.png width=100% />

I have already done the partition step for you (by splitting up the tweets into one file per day).
You will have to do the map and reduce steps.

## Background Tasks

Complete the following tasks to familiarize yourself with the sample code:

1. Fork the [twitter\_coronavirus](https://github.com/mikeizbicki/twitter_coronavirus) repo and clone it onto the lambda server.

1. **Mapping:**
   The `map.py` file processes the zip file for an individual day.
   From the root directory of your clone, run the command
   ```
   $ ./src/map.py --input_path=/data/Twitter\ dataset/geoTwitter20-02-16.zip
   ```
   This command will take a few minutes to run as it is processing all of the tweets within the zip file.
   After the command finishes, you will now have a folder `outputs` that contains a file `/geoTwitter20-02-16.zip.lang`.
   This is a file that contains JSON formatted information summarizing the tweets from 16 February.

1. **Visualizing:**
   The `visualize.py` file displays the output from running the `map.py` file.
   Run the command
   ```
   $ ./src/visualize.py --input_path=outputs/geoTwitter20-02-16.zip.lang --key='#coronavirus'
   ```
   This displays the total number of times the hashtag `#coronavirus` was used on 16 February in each of the languages supported by twitter.
   Now manually inspect the output of the `.lang` file using vim:
   ```
   $ vim outputs/geoTwitter20-02-16.zip.lang
   ```
   You should see that the file contains a dictionary of dictionaries.
   The outermost dictionary has languages as the keys, 
   and the innermost dictionary has hashtags as the keys.
   The `visualize.py` file simply provides a nicer visualization of these dictionaries.

1. **Reducing:**
   The `reduce.py` file merges the outputs generated by the `map.py` file so that the combined files can be visualized.
   Generate a new output file by running the command
   ```
   $ ./src/map.py --input_path=/data/Twitter\ dataset/geoTwitter20-02-17.zip
   ```
   Then merge these output files together by running the command
   ```
   $ ./src/reduce.py --input_paths outputs/geoTwitter20-02-16.zip.lang outputs/geoTwitter20-02-17.zip.lang --output_path=reduced.lang
   ```
   Alternatively, you can use the glob to merge all output files with the command
   ```
   $ ./src/reduce.py --input_paths outputs/geoTwitter*.lang --output_path=reduced.lang
   ```
   Now you can visualize the `reduced.lang` file with the command
   ```
   $ ./src/visualize.py --input_path=reduced.lang --key='#coronavirus'
   ```
   and this displays the combined result.

## Tasks

Complete the following tasks:

**Task 1: Map**

Modify the `map.py` file so that it tracks the usage of the hashtags on both a language and country level.
This will require creating a variable `counter_country` similar to the variable `counter_lang`, 
and modifying this variable in the `#search hashtags` section of the code appropriately.
The output of running `map.py` should be two files now, one that ends in `.lang` for the language dictionary (same as before),
and one that ends in `.country` for the country dictionary.

> **HINT:**
> Most tweets contain a `place` key,
> which contains a dictionary with the `country_code` key.
> This is how you should lookup the country that a tweet was sent from.
> Some tweets, however, do not have a `country_code` key.
> This can happen, for example, if the tweet was sent from international waters or the [international space station](https://unistellaroptics.com/10-years-ago-today-the-first-tweet-was-sent-directly-from-space/).
> Your code will have to be generic enough to handle edge cases similar to this without failing.

**Task 2: Reduce**

Once your `map.py` file has been modified to track results for each country,
you should run the map file on all the tweets in the `/data/Twitter\ dataset` folder from 2020.

> **HINT:**
> Use the glob `*` to select only the tweets from 2020 and not all tweets.

You should create a shell script `run_maps.sh` that loops over each file in the dataset and runs `map.py` on that file.
Each call to `map.py` can take up to a day to finish, so you should use the `nohup` command to ensure the program continues to run after you disconnect and the `&` operator to ensure that all `map.py` commands run in parallel.

After your modified `map.py` has run on all the files,
you should have a large number of files in your `outputs` folder.
Use the `reduce.py` file to combine all of the `.lang` files into a single file,
and all of the `.country` files into a different file.

**Task 3: reduce**

Recall that you can visualize your output files with the command
```
$ ./src/visualize.py --input_path=PATH --key=HASHTAG
```
Currently, this prints the top keys to stdout.

Modify the `visualize.py` file so that it generates a bar graph of the results and stores the bar graph as a png file.
The horizontal axis of the graph should be the keys of the input file,
and the vertical axis of the graph should be the values of the input file.
The final results should be sorted from low to high, and you only need to include the top 10 keys.

> **HINT:**
> We are not covering how to create images from python code in this class.
> I recommend you use the matplotlib library,
> and you can find some samples to base your code off of [in the documentation here](https://matplotlib.org/3.1.1/tutorials/introductory/sample_plots.html).

Then, run the `visualize.py` file with the `--input_path` equal to both the country and lang files created in the reduce phase, and the `--key` set to `#coronavirus` and `#코로나바이러스`.
This should generate four plots in total.

**Task 4: uploading**

Commit all of your code and images output files to your github repo and push the results to github.
You must edit the `README.md` file to provide a brief explanation of your results and include the 4 generate png files.
This explanation should be suitable for a future employer to look at while they are interviewing you to get a rough idea of what you accomplished.
(And you should tell them about this in your interviews!)

## Submission

Upload a link to you github repository on sakai.
I will look at your code and visualization to determine your grade.

Notice that we are not using CI to grade this assignment.
There's two reasons:

1. You can get slightly different numbers depending on some of the design choices you make in your code.
    For example, should the term `corona` count tweets that contain `coronavirus` as well as tweets that contain just `corona`?
    These are relatively insignificant decisions.
    I'm more concerned with your ability to write a shell script and use `nohup`, `&`, and other process control tools effectively.
