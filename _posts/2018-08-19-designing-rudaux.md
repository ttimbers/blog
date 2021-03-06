---
layout: post
title: Designing Rudaux
date: 2018-08-19
excerpt: The development of Canvas and JupyterHub Integration in UBC's Data Science 100.
heroImage: paint-strokes.jpg
heroColor: '#525659'
imageAuthor: Samuel Zeller
imageLink: https://unsplash.com/@samuelzeller
---

<!-- close content tag -->
</div>

<div class="card post-info">
  <div class="card-content">
    <div class="content">
      <blockquote>
        This post is focused on the motivation and design process in building Rudaux. For information on how to use Rudaux to integrate Canvas and JupyterHub, please read <a href="using-rudaux"><em>Using Rudaux</em></a>.
      </blockquote>
    </div>
  </div>
  <footer class="card-footer">
    <a href='https://samhinshaw.github.io/rudaux-docs/' class='card-footer-item'>
      <span class="icon is-medium">
        <i class="fas fa-book fa-lg"></i>
      </span>
      <span class='link-description'>Documentation</span>
    </a>
    <a href='http://github.com/samhinshaw/rudaux' class='card-footer-item'>
      <span class="icon is-medium">
        <i class="fab fa-github fa-lg"></i>
      </span>
      <span class='link-description'>Source Code</span>
    </a>  
  </footer>
</div>

<!-- resume content tag -->
<div class="content">

<!-- > This post is focused on the motivation and design process in building Rudaux. For information on how to use Rudaux to integrate Canvas and JupyterHub, please read _[Using Rudaux](../using-rudaux)_. -->

<h2 id='motivation'>Motivation</h2>

Rudaux was designed to be an interface for course administration that automates away a lot of the tedious functions that an instructor might otherwise have to perform. These functions are vital to keep the course running smoothly but takes valuable from the instructor during the semester. Some examples of these tasks include:

- Managing assignments in the learning management system.
- Grading assignments.
- Return grades to students.

For its initial release, Rudaux was designed expressly with the [UBC's new Data Science 100 course](https://github.com/UBC-DSCI/dsci-100) by [Tiffany Timbers](https://twitter.com/TiffanyTimbers) in mind. This course is aimed at students with no prior computing experience yet students are required to complete all assignments in R using Jupyer notebooks. To accomplish this using standard course operating procedures (students install and run software on their own machine, submit Jupyer notebook files for homework that TAs then have to run/test on their computers to grades, etc) would really limit the scalibility of the course as well as distract students from their course learning goals. 

Therefore, with this in mind we decided upon using Canvas with JupyterHub for our teaching platform, along with nbgrader for grading. This created the need for tools which would leverage Instructure's powerful API for Canvas to automate a great deal of course administration requirements. This automation would allow the instructor to focus on teaching, while maintaining many of the benefits that integration of Canvas & JupyterHub & nbgrader provide to students. In particular, it means Canvas becomes the single web address students need to think about for all of their course needs. From there, they can view their assignments, launch them in JupyterHub, and receive feedback on their graded assignments, all in one place.

## Infrastructure

<h3 id='canvas'>Canvas</h3>

[Canvas](https://www.canvaslms.com/) was the ideal choice for our Learning Management System (LMS). The University of British Columbia (UBC) has recently adopted Canvas as their LMS in place of Connect (the system previously used) and as such there is a great deal of support for the Canvas LMS at our institution. Furthermore, Canvas is a widely used LMS that also uses and supports Learning Tools Interoperability (LTI) which is a standard method of linking/connecting a LMS to external service tools (e.g., iClicker, LockDown Browser, Piazza, etc). Importantly, a [LTI authenticator was recently developed for JupyterHub](https://github.com/jupyterhub/ltiauthenticator) and thus it is now possible to connect Canvas and JupyterHub using LTI. Although, to our knowledge this had never been done before. However, JupyterHub's LTI authenticator had been tested and used with the EdX LMS for a Spring 2018 [online offering of Berkely's Data8 course](https://www.edx.org/course/foundations-data-science-computational-uc-berkeleyx-data8-1x). 

<h4 id='lti-authentication'>LTI Authentication</h4>

<!-- close content tag -->
</div>

<article class="message">
  <div class="message-header" id='lti-definition-header'>
    <p>
      LTI Terminology
      <small>(click to show)</small>
    </p>
    <span class="icon">
      <i class="fas fa-chevron-down"></i>
    </span>
  </div>
  <div class="message-body" id='lti-definition-body' style="display: none;">
    <div class="content">
      <dl>
        <dt><span>LTI Consumer:</span></dt>
        <dd>The service sending the launch request. Usually this will be your LMS. In our case, this refers to Canvas.</dd>
        <dt><span>LTI Tool Provider:</span></dt>
        <dd>The service receiving the launch request, and 'providing the service'. In our case this is JupyterHub.</dt>
        <dt><span>LTI Consumer Key:</span></dt>
        <dd>A long randomly generated hex string that serves as the first half of our authentication token. This is similar to a username or a public key.</dd>
        <dt><span>LTI Consumer Secret:</span></dt>
        <dd>A long randomly generated hex string that serves as the second half of our authentication token. This is similar to a password or a private key.</dd>
        <dt><span>LTI User ID:</span></dt>
        <dd>An anonymous user ID generated by the LTI consumer.</dd>
      </dl>
    </div>
  </div>
</article>

<!-- resume content tag -->
<div class='content'>

To make this happen, we are using the [ltiauthenticator module](https://github.com/jupyterhub/ltiauthenticator) for JupyterHub written by Yuvi Panda ([@yuvipanda](https://twitter.com/yuvipanda)). This module receives LTI launch requests from LTI consumers such as Canvas and passes the user's ID to JupyterHub as a username. If the launch request contains a Canvas ID parameter, it uses that as the user's ID. Otherwise, it uses the LTI User ID (see '[Course Privacy](#course-privacy)' for more information).

For more detailed information on setting up LTI authentication between Canvas and JupyterHub, please read the [ltiauthenticator documentation](https://github.com/jupyterhub/ltiauthenticator#canvas).

<h3 id='github-repositories'>GitHub Repositories</h3>

Borrowing from the [UBC Master of Data Science](https://ubc-mds.github.io/) course management model, and to maximize compatibility with nbgrader, we chose to store all of our course documents in one private GitHub repository, and a subset of these&mdash;the student lab/assignment files where the solutions have been removed in a public repository.

We refer to the first, private repository as our **instructors' repository**. In this repository we have the master copies of the assignments (the `source/` step of nbgrader), which contains the solutions as well as the tests. We also have the `gradebook.db` SQLite database checked-in to version control. We intend to implement a more appropriate solution for managing this in the future, but for the first run of the course, we believe it to be an acceptable solution.

Our second, public repository only contains the student copies of the assignments (the `release/` step of nbgrader). We refer to this as the **students' repository**. Each link we provide in Canvas (which the students see and access as an assignment button) has a query string attached which triggers a program, called nbgitpuller, to sync the student's home directory to this repository and redirect them to the assignment's notebook. nbgitpuller is a very nicely designed tool which is able to redirect the student to the appropriate version of the file they should be working on. For example, the first time the student accesses the notebook it pulls it from the public GitHub students' repository and stores a copy for that Student on the JupyterHub server. The next time the student goes to work on the same notebook, nbgitpuller is able to redirect the student to copy on the server that they were previously working on. Additionally, if the Instructor needs to send updates/changes to the the public GitHub students' repository copy, the next time the student goes to that notebook through Canvas, nbgitpuller will try to pull and merge those changes. 


<h3 id='jupyterhub-servers'>JupyterHub Servers</h3>

We use two JupyterHub servers to administer DSCI 100. These virtual machines for these servers are provisioned with Terraform, and set up with Ansible. [Ian Allison](https://github.com/ianabc) put in a tremendous amount of work setting up these servers and making their deployments programmatic and reproducible. All of the code for setting up these servers is available in our [infrastructure repository](https://github.ubc.ca/UBC-DSCI/dsc100-infra).

<h4 id='student-server'>Student Server</h4>

The first Jupyterhub server is dedicated solely to student use. Students log in by clicking on a LTI-enabled link in Canvas (looks like an assignment button to the students), and are authenticated and redirected to the notebook for that assignment. Using [dockerspawner](https://github.com/jupyterhub/dockerspawner), a docker container is spawned for each user, and their home directory is mapped to a folder on a [ZFS fileserver](#fileserver). The assignment is stored on the server persistantly, as is the students work. The students can revisit the notebook and see and edit their work as many times as they would like through the link in Canvas. 

<h4 id='grading-server'>Grading Server</h4>

By contrast, the grading server is only accessible to the instructor and TAs. This JupyterHub server uses [Shibboleth authentication](<https://en.wikipedia.org/wiki/Shibboleth_(Shibboleth_Consortium)>) for login and access control with UBC credentials. The grading is not done on the same server that students are using, as nbgrader can be quite resource-intensive, and we do not wish to degrade students' experience.

We chose to use nbgrader as the software for grading for several reasons:

1. It supports automated grading for code questions (and we also untilize it for multiple choice questions by storing the answer as a variable). And students can see and run the tests for autograded questions as many times as they would like to work toward the correct solution.

2. It has a command line tool, as well as an API so that it can be used in a very automated fashion.

3. It works with R (the language we are using for DSCI 100) in Jupyter notebooks, as well as Python (and probably any other language used in Jupyter). 

4. It allows seamless integration of manual grading along with the autograded answers (we believe that some of the students work should be seen by human eyeballs and given human feedback).

5. It has a nice GUI for TA's to perform manual grading and inline feedback.

6. It generates a lovely HTML report for the students with grades and feedback inline with the questions.

<h4 id='fileserver'>ZFS fileserver</h4>

The directory structure of is roughly thus, where `/tank/home` is the mount point of the fileserver:

```sh
/tank/home
└── dsci100
    ├── canvas-user-id-1
    │   └── student-repo-name
    │       └── materials
    │           ├── assignment1
    │           └── assignment2
    └── canvas-user-id-2
        └── student-repo-name
            └── materials
                ├── assignment1
                └── assignment2
```

While the fileserver does not need to run a ZFS filesystem, it is advantageous for its ability to snapshot the filesystem. This allows us to take a snapshot at the exact time an assignment is due and copy from that snapshot without worry about files changing during copying.

## Rudaux

Rudaux was designed to harmonize these pieces of infrastructure. For an overview of its main functions, please see _[Using Rudaux](using-rudaux)_, or [rudaux's documentation](https://samhinshaw.github.io/rudaux-docs/).

Rudaux consists of three main components:

1. A `Course` class, which facilitates operations on an entire course.
2. An `Assignment` class, which facilitates operations on individual assignments.
3. A command line interface, which parses common commands and runs the associated python code, without the need for the instructor to write a python script.

<h2 id='reflections'>Reflections</h2>

One of the decisions I had to make when building rudaux was where to put various functions--as a Course method or an Assignment method. One such example was the `assign()` function. At first, it might seem to be an obvious fit for an Assignment method. However, when assigning, it was necessary to clone the students' repository, copy the assigned versions of the assignment to it, and then commit and push the results. It would be wildly inefficient to do this multiple times when assigning multiple assignments in a row. Therefore, while it may have been possible to engineer a tricky solution to keep this method on the Assignment object, for simplicity I moved it to the Course object.

Another obstacle was where best to store state. I could store state within each class, but it would not persist through multiple script calls. Worried about keeping state in sync, for the initial release, I have opted to not persist state, and simply re-fetch fresh information at runtime.

<h2 id='looking-forward'>Looking Forward</h2>

In future, most of the features I would like to implement for rudaux would be abstractions that remove some of [the assumptions that rudaux currently operates on](https://samhinshaw.github.io/rudaux-docs/config/#assumptions), making it applicable to more use cases. For example, it would be nice to be able to run rudaux from any location, scheduling cron jobs on the grading server remotely. Additionally, it would be great to abstract some of the git repo management, offering greater flexibility in course setup.

Furthermore, I would like to contribute functionality back to the nbgrader API. Where functionality was lacking, I dug into the nbgrader source code and copied the relevant portions into rudaux. I have [already contributed a `feedback()` function](https://github.com/jupyter/nbgrader/pull/1003) to the nbgrader API. Ideally I would also like to add a function to the gradebook which returns a student's grade for a given assignment.

## Your Thoughts

What do you think? We'd love to hear your thoughts on rudaux and your potential use cases. Please visit the [rudaux issues](https://github.com/samhinshaw/rudaux/issues) to get started. We would also love others to try out rudaux and contribute to its growth and development. 

You can [contact me](https://twitter.com/samhinshaw), or the course's creator, [Dr. Tiffany Timbers](https://twitter.com/tiffanytimbers) on twitter.
