# auto-cancel-workflow-orb

This is a CircleCI orb for canceling all jobs in a workflow when one of a job in the workflow is failed.
This orb uses v2 CircleCI API to get the status of jobs and cancel all jobs.

## Motivation
CircleCI has `Auto-cancel redundant builds` feature. This feature automatically cancels any queued or running builds when there is a build already running on the same branch. However, this orb function is a different.

It's used in the same use case as `requires` in workflow. This is to reduce the build time.
For example, there are 4 jobs in a workflow, and 3 jobs run concurrently and then run 1 job. In this case, the previous 3 jobs must all succeed in order to run the last job. If 1 of 3 jobs is failed, it doesn't need to run the final job like deploy. So, we want to cancel all jobs in the workflow when any one of these was failed.

The below example, the total build time is `15 mins`, but once `job_3` is failed, we know it doesn't need to wait for `job_1` and `job_2`. But if we set the `requires` option for each job, the total build time will be increasing. So, I decided to create this orb.

```
workflows:
  version: 2
  workflow:
    jobs:
      - job-1 (run 10 mins && success)
      - job-2 (run 15 mins && success)
      - job-3 (run 1 mins && failed)
      - deploy:
          requires:
            - job-1
            - job-2
            - job-3
```
