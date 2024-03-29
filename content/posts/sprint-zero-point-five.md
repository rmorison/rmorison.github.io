+++
title = "Sprint 0.5: Planning a New Service Build"
author = ["Rod Morison"]
date = 2022-08-27T01:06:00-07:00
aliases = ["/posts/sprint-0"]
draft = false
topics = ["Software architecture", "Project management"]
description = "Sprint 0.5: uncovering the hidden work in a new service build"
+++

## The Sprint 0.5 Scenario {#the-sprint-0-dot-5-scenario}

I often come across new, [greenfield](https://en.wikipedia.org/wiki/Greenfield_project) backend projects or [brownfield](https://en.wikipedia.org/wiki/Brownfield_(software_development)) rewrites of an existing service (usually a monolith) into a new service architecture (usually microservice), and a new infrastructure (cloud migration). It's common---and a best practice---to kick off such a project with a [discovery phase](https://en.wikipedia.org/wiki/Application_discovery_and_understanding), unless product requirements are already developed and well understood by the implementation team. The discovery phase is sometime called "Sprint 0", though it's often a much bigger time and effort investment than a common development sprint.

It's also common to invest in some technical architecture and project planning during that discovery sprint. What I've observed in some projects is the technical planning in discovery is just enough for a modicum of work breakdown and estimation. Discovery typically delivers just enough design documentation for feature build tickets or story tasks. However, and especially for a greenfield build, all that planning leaves a host of implementation questions hanging.

Are we set on language, libraries, and web stack? Have we agreed on code formatters, linters, static checkers? What are we using for CI/CD? Do we have a devops plan, to write infrastructure code and bring up multiple environments? And so on.

Sometimes, when an experienced team implements in a familiar organization with established practices, these questions are mostly answered. More often, work such as this creates huge friction in the early sprints and divergence across teams working on different services. And even for the ready-to-roll team, it's worth going through a checklist, look for upgrade opportunities.

The pressure to, "Get going with delivery", can be heavy.  But resist!…if there's a load of these non-recurring tasks hanging unresolved. Get them scheduled in, with full visibility.

I call this _Sprint 0.5_ work: [NRE](https://en.wikipedia.org/wiki/Non-recurring_engineering) that has to be done, is better done at the beginning, and creates friction, disorganization, and divergence complexity of not done up front.


## Refine the Architecture, Revisit the Plan {#refine-the-architecture-revisit-the-plan}

When discovery phase technical architecture, planning and estimation is done, I believe it's worth coming back to at the end of Sprint 0.5. The team has a lot more "feel" for the implementation at this point and is likely to do a deeper dive into tickets and more accurate estimates.

Or, better yet, only do the top level architecture and project breakdown, with minimal [t-shirt size](https://asana.com/resources/t-shirt-sizing) accuracy estimation during discovery. Save the deep dive until the tech work of Sprint 0.5 is done.

Of course, commitment to stakeholders, budget and deadline pressure may make this untenable. But at least validate your discovery project plan and estimates. And if Sprint 0.5 estimates are significantly greater than those from discovery, well that's another problem.
![](/ox-hugo/brown_and_black_turtle_on_brown_sand-scopio-01f822e0-47d5-4952-b881-2331aa99c598.jpg)


## Sprint 0.5 Project Breakdown {#sprint-0-dot-5-project-breakdown}

Here I offer a canonical work breakdown for a sprint 0.5. The epics breakdown are part of a more detailed breakdown in this [Sprint 0.5 project breakdown](https://docs.google.com/spreadsheets/d/1VmFfYiROtW4wktL4Exlz9NODEgOy4tTJb9lJS5lP9EE/edit?usp=sharing) Google sheet.

<!-- This HTML table template is generated by emacs/table.el -->
<table border="1">
  <tr>
    <td align="left" valign="top">
      Epic&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      Tasks&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  <tr>
    <td align="left" valign="top">
      Service&nbsp;stack&nbsp;template(s)&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      Select&nbsp;language,&nbsp;libraries,&nbsp;web&nbsp;stack&nbsp;for&nbsp;service&nbsp;and&nbsp;serverless<br />
      implementations;&nbsp;build&nbsp;out&nbsp;a&nbsp;"cookiecutter"&nbsp;template&nbsp;project;&nbsp;&nbsp;&nbsp;<br />
      add&nbsp;standardized&nbsp;test&nbsp;runner,&nbsp;coverage,&nbsp;linters,&nbsp;static&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      checkers,&nbsp;and&nbsp;formatters.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  <tr>
    <td align="left" valign="top">
      Continuous&nbsp;integration&nbsp;(CI)<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      Decide&nbsp;on&nbsp;CI&nbsp;(Jenkins,&nbsp;GH&nbsp;Actions,&nbsp;BB&nbsp;Pipelines,&nbsp;Travis,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      CircleCI);&nbsp;setup&nbsp;servers&nbsp;and/or&nbsp;runners;&nbsp;containerize&nbsp;template&nbsp;&nbsp;<br />
      service(s);&nbsp;add&nbsp;test,&nbsp;lint,&nbsp;format&nbsp;to&nbsp;CI;&nbsp;add&nbsp;Dockerfile,&nbsp;CI&nbsp;to&nbsp;<br />
      templates&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  <tr>
    <td align="left" valign="top">
      Cloud&nbsp;infrastructure&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      Build&nbsp;an&nbsp;environment&nbsp;in&nbsp;infrastructure&nbsp;with&nbsp;template&nbsp;service(s):<br />
      container&nbsp;strategy&nbsp;(orchestration,&nbsp;etc),&nbsp;networking,&nbsp;databases,&nbsp;<br />
      proxies,&nbsp;load&nbsp;balancers;&nbsp;implement&nbsp;with&nbsp;infrastructure&nbsp;code&nbsp;and&nbsp;<br />
      validate&nbsp;with&nbsp;env&nbsp;spinup&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  <tr>
    <td align="left" valign="top">
      Continuous&nbsp;deployment&nbsp;(CD)&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      Establish&nbsp;branch/release/env&nbsp;SDLC&nbsp;strategy;&nbsp;connect&nbsp;service&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      builds&nbsp;with&nbsp;env&nbsp;deploys&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  <tr>
    <td align="left" valign="top">
      Logging&nbsp;&&nbsp;monitoring&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      Setup&nbsp;logging,&nbsp;APM,&nbsp;trace&nbsp;in&nbsp;service&nbsp;stack&nbsp;and&nbsp;feed&nbsp;to&nbsp;central&nbsp;&nbsp;<br />
      logging;&nbsp;setup&nbsp;service&nbsp;health&nbsp;checks;&nbsp;add&nbsp;anomaly&nbsp;alerting&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      (Slack,&nbsp;SMS,&nbsp;Pagerduty)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  <tr>
    <td align="left" valign="top">
      Service&nbsp;architecture&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      Design&nbsp;entities,&nbsp;APIs,&nbsp;async&nbsp;tasks,&nbsp;etc.&nbsp;for&nbsp;initial&nbsp;set&nbsp;of&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      services,&nbsp;including&nbsp;registration/authentication&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  <tr>
    <td align="left" valign="top">
      Planning&nbsp;&&nbsp;estimating&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      Use&nbsp;architecture&nbsp;level&nbsp;design&nbsp;for&nbsp;implementation&nbsp;stories,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br />
      estimates,&nbsp;and&nbsp;scheduling&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  <tr>
    <td align="left" valign="top">
      Initial&nbsp;build:&nbsp;sprint&nbsp;1,2,…<br />
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
    <td align="left" valign="top">
      At&nbsp;this&nbsp;point&nbsp;each&nbsp;service&nbsp;can&nbsp;drop&nbsp;a&nbsp;code&nbsp;template&nbsp;with&nbsp;CI/CD&nbsp;&nbsp;<br />
      ready&nbsp;to&nbsp;go&nbsp;and&nbsp;start&nbsp;a&nbsp;sequence&nbsp;of&nbsp;delivery&nbsp;sprints&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
</table>


## A Sprint 0.5 Gantt Chart {#a-sprint-0-dot-5-gantt-chart}

As a practical matter of efficiency, much of Sprint 0.5 can be split among software architects, developers, and devops staff, allowing work in parallel, as suggested below.

{{< figure src="/ox-hugo/sprint-0.5-gantt.png" >}}
