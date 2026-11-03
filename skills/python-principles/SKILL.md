---
name: python-principles [WIP]
description: How to construct and compose small functions and operations into a larger program 
---

# Definitions

- function: small function that does one task
- pipeline: a composition of functions to perform a larger task
- program: a collection of pipelines that can be used for a larger use-case

Do things once, reuse later, 

## pipeline

Example:
Given a dataset we want to filter it, combine some items, then generate something with an LLM

def read_data() -> list[...]:
  ... # loading of data

def filter(x: list[...]) -> list[...]:
  ... # apply some filters

def combiner(x: list[...]) -> list[...]:
  ... # combine some elements

def generater(x: list[...]) -> list[...]:
  ... # do stuff
