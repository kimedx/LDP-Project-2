# Large Scale Data Processing: Project 2

Worked alone | Using first late-day policy for 24-hour extension

## Results for Submission
### Part 1 (exactF2)
| Machine | F_2 | Time Elapsed |
|---------|---------|--------------|
| Local | 8,567,966,130 | 26s |
| GCP | 8,567,966,130 | 83s |

### Part 2 (Tug-of-War)
| Machine | F_2 | Time Elapsed | Additional Input Values |
|---------|---------|--------------|-------------------------|
| Local | 9,231,024,631 | 130s | width = 10, depth = 3 |
| GCP | 8,288,498,975 | 297s | width = 10, depth = 3 |

### Part 3 (BJKST)
| Machine | F_0 | Time Elapsed | Additional Input Values |
|---------|---------|--------------|-------------------------|
| Local | 7,372,800 | 7s | width = 256, trials = 5 |
| GCP | 7,733,248 | 40s | width = 256, trials = 5 |

### Part 4-1 (BJKST (width): 256 depth: 5)vs. exactF0)
| Algorithm | Machine | F_0 | Time Elapsed |
|-----------|---------|-----|--------------|
| BJKST | Local | 7,372,800 | 7s | 
| exactF0 | Local | 7,406,649 | 25s |
| BJKST | GCP | 7,733,248 | 40s | 
| exactF0 | GCP | 7,406,649 | 67s |

Analysis: While both BJKST algorithm runs completed in less than 40 seconds both on the local machine and on GCP, the algorithm that computes the exact F0 value took 67 seconds on GCP, providing evidence for the efficiency of BJKST algorithm. The precision of the estimations was also reasonable, with local result being 0.46% off from the exact value and the GCP result being 4.4% off from the exact value.

### Part 4-2 (Tug-of-War(width:10 depth:3) vs. exactF2)
| Algorithm | Machine | F_2 | Time Elapsed |
|-----------|---------|-----|--------------|
| Tug-of-War | Local | 9,231,024,631 | 130s |
| exactF2 | Local | 8,567,966,130 | 26s |
| Tug-of-War | GCP | 8,288,498,975 | 297s | 
| exactF2 | GCP | 8,567,966,130 | 83s |

Analysis: Unlike the comparison between the BJKST algorithm and the exactF0 algorithm, Tug-of-War sketch turned out to be significantly slower than the exactF2 algorithm on both the local machine and on GCP. Regarding precision, the Tug-of-War algorithm showed reliable precision being 7.74% off from the actual value on local machine and 3.27% off on GCP.

## Getting started
Head to [Project 1](https://github.com/CSCI3390Spring2025/project_1) if you're looking for information on Git, template repositories, or setting up your local/remote environments.

## Resilient distributed datasets in Spark
This project will familiarize you with RDD manipulations by implementing some of the sketching algorithms the course has covered thus far.  

You have been provided with the program's skeleton, which consists of 5 functions for computing either F0 or F2: the BJKST, tidemark, tug-of-war, exact F0, and exact F2 algorithms. The tidemark and exact F0 functions are given for your reference.

## Relevant data

You can find the TAR file containing `2014to2017.csv` [here](https://drive.google.com/file/d/1MtCimcVKN6JrK2sLy4GbjeS7E2a-UMA0/view?usp=sharing). Download and expand the TAR file for local processing. For processing in the cloud, refer to the steps for creating a storage bucket in [Project 1](https://github.com/CSCI3390Spring2025/project_1) and upload `2014to2017.csv`.

`2014to2017.csv` contains the records of parking tickets issued in New York City from 2014 to 2017. You'll see that the data has been cleaned so that only the license plate information remains. Keep in mind that a single car can receive multiple tickets within that period and therefore appear in multiple records.  

**Hint**: while implementing the functions, it may be helpful to copy 100 records or so to a new file and use that file for faster testing.  

## Calculating and reporting your findings
You'll be submitting a report along with your code that provides commentary on the tasks below.  

1. **(3 points)** Implement the `exact_F2` function. The function accepts an RDD of strings as an input. The output should be exactly `F2 = sum(Fs^2)`, where `Fs` is the number of occurrences of plate `s` and the sum is taken over all plates. This can be achieved in one line using the `map` and `reduceByKey` methods of the RDD class. Run `exact_F2` locally **and** on GCP with 1 driver and 4 machines having 2 x N1 cores. Copy the results to your report. Terminate the program if it runs for longer than 30 minutes.
2. **(3 points)** Implement the `Tug_of_War` function. The function accepts an RDD of strings, a parameter `width`, and a parameter `depth` as inputs. It should run `width * depth` Tug-of-War sketches, group the outcomes into groups of size `width`, compute the means of each group, and then return the median of the `depth` means in approximating F2. A 4-universal hash function class `four_universal_Radamacher_hash_function`, which generates a hash function from a 4-universal family, has been provided for you. The generated function `hash(s: String)` will hash a string to 1 or -1, each with a probability of 50%. Once you've implemented the function, set `width` to 10 and `depth` to 3. Run `Tug_of_War` locally **and** on GCP with 1 driver and 4 machines having 2 x N1 cores. Copy the results to your report. Terminate the program if it runs for longer than 30 minutes. **Please note** that the algorithm won't be significantly faster than `exact_F2` since the number of different cars is not large enough for the memory to become a bottleneck. Additionally, computing `width * depth` hash values of the license plate strings requires considerable overhead. That being said, executing with `width = 1` and `depth = 1` should generally still be faster.
3. **(3 points)** Implement the `BJKST` function. The function accepts an RDD of strings, a parameter `width`, and a parameter `trials` as inputs. `width` denotes the maximum bucket size of each sketch. The function should run `trials` sketches and return the median of the estimates of the sketches. A template of the `BJKSTSketch` class is also included in the sample code. You are welcome to finish its methods and apply that class or write your own class from scratch. A 2-universal hash function class `hash_function(numBuckets_in: Long)` has also been provided and will hash a string to an integer in the range `[0, numBuckets_in - 1]`. Once you've implemented the function, determine the smallest `width` required in order to achieve an error of +/- 20% on your estimate. Keeping `width` at that value, set `depth` to 5. Run `BJKST` locally **and** on GCP with 1 driver and 4 machines having 2 x N1 cores. Copy the results to your report. Terminate the program if it runs for longer than 30 minutes.
4. **(1 point)** Compare the BJKST algorithm to the exact F0 algorithm and the tug-of-war algorithm to the exact F2 algorithm. Summarize your findings.

## Submission via GitHub
Delete your project's current **README.md** file (the one you're reading right now) and include your report as a new **README.md** file in the project root directory. Have no fear—the README with the project description is always available for reading in the template repository you created your repository from. For more information on READMEs, feel free to visit [this page](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-readmes) in the GitHub Docs. You'll be writing in [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown). Be sure that your repository is up to date and you have pushed all changes you've made to the project's code. When you're ready to submit, simply provide the link to your repository in the Canvas assignment's submission.

## You must do the following to receive full credit:
1. Create your report in the ``README.md`` and push it to your repo.
2. In the report, you must include your (and your group members') full name in addition to any collaborators.
3. Submit a link to your repo in the Canvas assignment.

## Late submission penalties
Please refer to the course policy.
