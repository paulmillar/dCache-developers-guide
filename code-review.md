# The code review process

This chapter will focus on the process of reviewing code, so that you understand what are the goals \(and what are not\) and how to approach the topic, both as someone submitting code for reviewing and someone undertaking code-review.

## Preparation: Setup and configuration of tools

We are using an instance of Review Board, located at https://rb.dcache.org. In order to interact with it, it is helpful to install Review Board's command line tools on your development machine. 

### Installing rbt

The rbt toolchain is available at https://www.reviewboard.org/downloads/rbtools/ and will require a recent Python installation on your machine.

As a very simple test, right after the installation, `rbt --version` should be callable from your development directory and return the version string of the version you just installed. 

### Configuring rbt

The `rbt` client needs connection information in order to know which RB instance to talk to. The configuration is documented at https://www.reviewboard.org/docs/rbtools/dev/rbt/configuration/repositories/#rbtools-repo-config. In short, call `rbt setup-repo` for an interactive way to generate the `.reviewboardrc` file from within your development directory. 

Make sure to edit the file afterwards to ensure your target branch property, `BRANCH`, is set to `master`. 

NB: The rc file should reside in the root directory of your local dCache repository (e.g., `~/devel/dcache/.reviewboardrc` is fine), not directly in your home directory as other dotfiles (`~/.reviewboardrc` is wrong).
