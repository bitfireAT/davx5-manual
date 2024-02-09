
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

First, install the required packages (these are the Ubuntu packages):

    apt install python3-sphinx python3-sphinx-rtd-theme

Also install the Youtube extension for sphinx:

    pip install sphinxcontrib-youtube

Then clone the repository and do your changes. To compile:

    sphinx-build . _build

Then you can see your compiled version in the `_build` directory.

When you have finished your changes and are satisifed with the preview, create a pull request with your changes.
