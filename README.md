
[![Check](https://github.com/bitfireAT/davx5-manual/actions/workflows/live.yml/badge.svg)](https://github.com/bitfireAT/davx5-manual/actions/workflows/live.yml)


DAVx⁵ Manual
============

[This repository](https://github.com/bitfireAT/davx5-manual) contains the
manual for [DAVx⁵](https://www.davx5.com) ([Github](https://github.com/bitfireAT/davx5-ose)).

Published version (updated automatically from main branch): https://www.davx5.com/manual/


Contributing
============

Thanks for your interest in contributing to this manual! To contribute:

1. Fork the repository.
1. Make the changes in the repo.
1. Send a pull request.

For your first contribution, our CLA bot will ask you to sign the Contributor License Agreement.


Working locally
===============

First, install the required packages (needs Python3 with venv support):

    python3 -mvenv .venv
    source .venv/bin/activate
    pip install -r requirements.txt

Then clone the repository and do your changes. To compile:

    make html

Then you can see your compiled version in the `_build` directory.

When you have finished your changes and are satisifed with the preview, create a pull request with your changes.
