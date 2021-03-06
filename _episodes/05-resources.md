---
title: "Using resources effectively"
teaching: 15
exercises: 30
questions:
- "How do I choose the correct resources for my jobs?"
- "How do I understand what resources my jobs are using?"
objectives:
- "Understand how perform basic benchmarking to choose resources efficiently."
keypoints:
- "To use resources effectively, you need to understand the performance of your jobs."
---

We now know virtually everything we need to know about getting stuff onto and using a cluster.
We can log on, submit different types of jobs, use preinstalled software, and install and use
software of our own. What we need to do now is use the systems effectively. To do this we need
to understand the basics of *benchmarking*. Benchmarking is essentially performing simple 
experiments to help understand how the performance of our work varies as we change the
properties of the jobs on the cluster - including input parameters, job options and resources used.

> ## Our example
> In the rest of this episode, we will use an example parallel application that sharpens
> an input image. Although this is a toy problem, it exhibits all the properties of a full
> parallel application that we are interested in for this course.
> 
> The main resource we will consider here is the use of compute core time as this is the
> resource you are usually charged for on HPC resources. However, other resources - such
> as memory use - may also have a bearing on how you choose resources and constrain your
> choice.
>
> 
> For those that have come across HPC benchmarking before, you may be aware that people
> often make a distinction between *strong scaling* and *weak scaling*:
> 
> - Strong scaling is where the problem size (i.e. the *application*) stays the same 
>   size and we try to use more cores to solve the problem faster.
> - Weak scaling is where the problem size increases at the same rate as we increase
>   the core count so we are using more cores to solve a larger problem.
> 
> Both of these approaches are equally valid uses of HPC. This example looks at strong scaling. 
{: .callout}

Before we work on benchmarking, it is useful to define some terms for the example we will
be using

  - **Program** The computer program we are executing (`sharpen` in the examples below)
  - **Application** The combination of computer program with particular input parameters
     (`sharpen` with `fuzzy.pgm` in our example below)

## Accessing the software and input

{% include /snippets/16/sharpen_details.snip %}

## Baseline: running in serial

Before starting to benchmark an application to understand what resources are best to use,
you need a *baseline* performance result. In more formal benchmarking, your baseline
is usually the minimum number of cores or nodes you can run on. However, for understanding
how best to use resources, as we are doing here, your baseline could be the performance on
any number of cores or nodes that you can measure the change in performance from.

Our `sharpen` application is small enough that we can run a serial (i.e. using a single core)
job for our baseline performance so that is where we will start

> ## Run a single core job
> Write a job submission script that runs the `sharpen` application on a single core. You
> will need to take an initial guess as to the walltime to request to give the job time 
> to complete. Submit the job and check the contents of the STDOUT file to see if the 
> application worked or not.
> 
> > ## Solution
> > 
> > Creating a file called `submit_sharpen.pbs`:
> > ```
{% include /snippets/16/serial_submit.snip %}
> > ```
> > {: .language-bash}
> > 
> > Submit with:
> > ```
> > {{ site.host_prompt }} {{ site.sched_submit }} submit_sharpen.pbs
> > ```
> > {: .language-bash}
> >
> > Output in STDOUT should look something like:
> >
> > ```
> > Input file is: fuzzy.pgm
> > Image size is 564 x 770
> > Using a filter of size 17 x 17
> > Reading image file: fuzzy.pgm
> > ... done
> > Starting calculation ...
> > On core 0
> > ... finished
> > Writing output file: sharpened.pgm
> > ... done
> > Calculation time was 5.400482 seconds
> > Overall run time was 5.496556 seconds
> > ```
> {: .solution}
{: .challenge}

Once your job has run, you should look in the output to identify the performance. Most 
HPC programs should print out timing or performance information (usually somewhere near
the bottom of the summary output) and `sharpen` is no exception. You should see two 
lines in the output that look something like:

```
Calculation time was 5.579000 seconds
Overall run time was 5.671895 seconds
```


{% include /snippets/16/view_output.snip %}

## Running in parallel and benchmarking performance

We have now managed to run the `sharpen` application using a single core and have a baseline
performance we can use to judge how well we are using resources on the system.

Note that we also now have a good estimate of how long the application takes to run so we can
provide a better setting for the walltime for future jobs we submit. Lets now look at how
the runtime varies with core count.

{% include /snippets/16/runtime_exercise.snip %}

## Understanding the performance

Now we have some data showing the performance of our application we need to try and draw some
useful conclusions as to what the most efficient set of resources are to use for our jobs. To
do this we introduce two metrics:

  - **Speedup** The ratio of the baseline runtime (or runtime on the lowest core count)
    to the runtime at the specified core count. i.e. baseline runtime divided by runtime
    at the specified core count.
  - **Ideal speedup** The expected speedup if the application showed perfect scaling. i.e. if
    you double the number of cores, the application should run twice as fast.
  - **Parallel efficiency** The percentage of *ideal speedup* actually obtained for a given
    core count. This gives an indication of how well you are exploiting the additional resources
    you are using.

We will now use our performance results to compute these two metrics for the sharpen application
and use the metrics to evaluate the performance and make some decisions about the most 
effective use of resources.

{% include /snippets/16/perf_exercise.snip %}

## Tips

Here are a few tips to help you use resources effectively and efficiently on HPC systems:

- Know what your priority is: do you want the results as fast as possible or are you happy
  to wait longer but get more research for the resources you have been allocated?
- Use your real research application to benchmark but try to shorten the run so you can turn
  around your benchmarking runs in a short timescale. Ideally, it should run for 10-30 minutes;
  short enough to run quickly but long enough so the performance is not dominated by static startup
  overheads (though this is application dependent). Ways to do this potentially include, for example:
  using a smaller number of time steps, restricting the number of SCF cycles, restricting the
  number of optimisation steps.
- Use basic benchmarking to help define the best resource use for your application. One 
  useful strategy: take the core count you are using as the baseline, halve the number of
  cores/nodes and rerun and then double the number of cores/nodes from your baseline and
  rerun. Use the three data points to assess your efficiency and the impact of different
  core/node counts.

{% include links.md %}
