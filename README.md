# You Don't Know Axios

> Promise based HTTP client for the browser and node.js

[axios][axios] is one of the most famous Javascript request libraries. According to Github's data, it is used by 2.1 millions repositories now (Feb 2020).

Purposes of this tutorial are,

- Clarifying misleading behaviors and usages
- Introducing design principles and internal implementations
- Helping users be able to solve problems by themselves
- Avoiding invalid or weak issues or pull requests opened
- Recording personal study notes
- Practicing English writing skills

I'd like to divide things into 3 questions,

- [Design Theories](#design-theories), why does axios look like this?
- [Usage Knowledges](#usage-knowledges), what didn't mention clearly in axios official document?
- [Problem Solutions](#problem-solutions), how to analyze a problem and open a good issue or pull request?

## Design Theories

### Keep Simple

The most impressive design in axios is its flexible architecture, including basic configs, interceptors, transformers and adapters. The core is simple and stable, while users can achieve customized functionalities by providing their own implementations. Before requesting for new features, ask yourself first that whether it is important and common enough to be added, or it can be solved by those hooks.

### Promise based

Make sure you are familiar with asynchronous programming when using axios, especially for Promise and async/await. Because axios connects internal things by Promise.

## Usage Knowledges

Let's follow the structure of official document. In each topic, I will give some examples to explain the misleading or unclear points.

### axios API

### Request Config

### Response Schema

### Config Defaults

### Interceptors

### Handling Errors

### Cancellation

## Problem Solutions


[axios]: https://github.com/axios/axios
