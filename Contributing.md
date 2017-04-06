## Contributing

### Writing new features, contributing source code

Please follow these guidelines so that your contribution can be integrated as easily as possible:

* Look through the [Issue list](http://github.com/leihs/leihs/issues) before you implement something, to make sure no one else is working on a similar feature.
* Create a fork of the leihs project.
* Base your changes off the `v4` branch.
* Make your changes in your own fork of leihs.
* Write tests for your work and ensure they pass. See the [[leihs-Developer-Guide]] for some hints on how to do that.
* Flatten your changes into one single commit. Push to your own fork.
* Submit a pull request.
* If our review approves it and if it applies cleanly, we'll run the whole test suite on our CI and in case all is green, merge it as soon as we can.

If you have any trouble, do not hesitate to create a new issue under the 'question' tag and we'll try to help you.

### General prerequisites for merging

Your code:

* Must not break any existing tests (checked by us on our CI server)
* Must satisfy our coding guidelines as verified by [Rubocop](https://github.com/bbatsov/rubocop), [Flay](https://github.com/seattlerb/flay) and [Flog](https://github.com/seattlerb/flog)
* Must be covered by new tests if it introduces new functionality

### Contributing translations

You can translate leihs into your language quite easily! See the [[leihs-Developer-Guide]] for instructions on how to create a translation.

### Sponsoring the development of features

You're not a programmer? No problem, your organization can sponsor a development company to create the features you need. Just look for a company that creates and maintains Ruby on Rails web applications. Any of them should be able to help you. Make sure to point the company at this guide so that they know how to bring their changes back into the project.
