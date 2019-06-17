---
name: Unexpected Problems / Bugs
about: Report unexpected problems about Tensorpack or its examples.

---

If you're asking about an unexpected problem which you do not know the root cause,
use this template. __PLEASE DO NOT DELETE THIS TEMPLATE, FILL IT__:

If you already know the root cause to your problem,
feel free to delete everything in this template.

### 1. What you did:

(1) **If you're using examples, what's the command you run:**

(2) **If you're using examples, have you made any changes to the examples? Paste `git status; git diff` here:**

(3) **If not using examples, tell us what you did:**

  It's always better to copy-paste what you did than to describe them.

  Please try to provide enough information to let others __reproduce__ your issues.
  Without reproducing the issue, we may not be able to investigate it.

### 2. What you observed:

(1) **Include the ENTIRE logs here:**

It's always better to copy-paste what you observed instead of describing them.

It's always better to paste **as much as possible**, although sometimes a partial log is OK.

Tensorpack typically saves stdout to its training log.
If stderr is relevant, you can run a command with `my_command 2>&1 | tee logs.txt`
to save both stdout and stderr to one file.

(2) **Other observations, if any:**
For example, CPU/GPU utilization, output images, tensorboard curves, if relevant to your issue.

### 3. What you expected, if not obvious.

If you expect higher speed, please read
http://tensorpack.readthedocs.io/tutorial/performance-tuning.html
before posting.

If you expect certain training results (e.g., accuracy), only in one of the two conditions can we help with it:
(1) You're unable to reproduce the results documented in tensorpack examples.
(2) It appears to be a tensorpack bug.

Otherwise, how to train a model is a machine learning question.
We do not answer machine learning questions and it is your responsibility to
figure out how to make your models more accurate.

### 4. Your environment:
  + Paste the output of this command: `python -c 'import tensorpack.tfutils as u; print(u.collect_env_info())'`
    If this command failed, tell us your version of Python/TF/tensorpack.
  + You can install Tensorpack master by `pip install -U git+https://github.com/ppwwyyxx/tensorpack.git`
    and see if your issue is already solved.
  + If you're not using tensorpack under a normal command line shell (e.g.,
    using an IDE or jupyter notebook), please retry under a normal command line shell.
  + Include relevant hardware information, e.g. number of GPUs used for training, amount of RAM.

You may often want to provide extra information related to your issue, but
at the minimum please try to provide the above information __accurately__ to save effort in the investigation.
