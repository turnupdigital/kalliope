# Order

- [Order](#order)
  - [Synopsis](#synopsis)
  - [Options](#options)
  - [Synapses example](#synapses-example)
    - [Normal order](#normal-order)
    - [Strict order](#strict-order)
    - [Ordered strict order](#ordered-strict-order)
    - [stt-correction](#stt-correction)
    - [stt-correction-file](#stt-correction-file)
    - [Use both stt-correction and stt-correction-file](#use-both-stt-correction-and-stt-correction-file)
    - [Order with arguments](#order-with-arguments)
  - [Notes](#notes)

## Synopsis

An **order** signal is a word, or a sentence caught by the microphone and processed by the STT engine.

## Options

| parameter | required | default | choices | comment                                             |
|-----------|----------|---------|---------|-----------------------------------------------------|
| order     | YES      |         |         | The order is passed directly without any parameters |

Other way to write an order, with parameters:

| parameter           | required | default | choices                        | comment                                                |
|---------------------|----------|---------|--------------------------------|--------------------------------------------------------|
| text                | YES      |         |                                | The order to match                                     |
| matching-type       | NO       | normal  | normal, strict, ordered-strict | Type of matching. See explanation bellow               |
| stt-correction      | NO       |         |                                | List of words from the order to replace by other words |
| stt-correction-file | NO       |         |                                | Same as stt-correction but load words from a YAML file |

**Matching-type:**
- **normal**: Will match if all words are present in the spoken order.
- **strict**: All word are present. No more word must be present in the spoken order.
- **ordered-strict**: All word are present, no more word and all word are in the same order as defined in the signal.

## Synapses example

### Normal order

Syntax:
```yml
signals:
  - order: "<sentence>"

signals:
  - order:
      text: "<sentence>"
      matching-type: "normal"
```

Example:
```yml
signals:
  - order: "please do this action"

signals:
  - order:
      text: "please do this action"
      matching-type: "normal"
```

In this example, with a `normal` matching type, the synapse would be triggered if the user say:
- please do this action
- please do this action with more word
- action this do please
- action this do please with more word

### Strict order

Syntax:
```yml
signals:
    - order:
        text: "<sentence>"
        matching-type: "strict"
```

Example:
```yml
signals:
    - order:
        text: "please do this action"
        matching-type: "strict"
```

In this example, with a `strict` matching type, the synapse would be triggered if the user say:
- please do this action
- action this do please

### Ordered strict order

Syntax:
```yml
signals:
    - order:
        text: "<sentence>"
        matching-type: "ordered-strict"
```

Example:
```yml
signals:
    - order:
        text: "please do this action"
        matching-type: "ordered-strict"
```

In this example, with a `strict` matching type, the synapse would be triggered if the user say:
- please do this action

### stt-correction

This option allow you to replace some words from the captured order by other word. 

Syntax:
```yml
signals:
    - order:
        text: "<sentence>"
        stt-correction:
          - input: "words to replace"
            output: "replacing words" 
```

E.g
```yml
- name: "stt-correction-test"
    signals:
      - order:
          text: "this is my order"
          stt-correction:
            - input: "test"
              output: "order"
    neurons:
      - debug:
          message: "hello"
```
In this example, if you pronounce "this is my test", the word test will be translated into "order" and so the signal "stt-correction-test" would b triggered.

This feature can be useful when working with numbers.
For example, you know that your STT engine return all number as string and you need them as integer for your neurons.

E.g:
```yml
- name: "mm-say"
    signals:
      - order:
          text: "this is my number {{ number }}"
          stt-correction:
            - input: "one"
              output: 1
    neurons:
      - debug:
          message: "{{ number }}"
```

In this example, if you say "this is my number one", Kalliope will translate the word "one" into "1".

### stt-correction-file

This option allow to set a list of corrections from a YAML file instead of writing them directly in the order.

Syntax:
```yml
signals:
    - order:
        text: "<sentence>"
        stt-correction-file: "<path to yaml file>"   
```

E.g
```yml
- name: "stt-correction-test"
    signals:
      - order:
          text: "this is my order"
          stt-correction-file: "my_stt_correction_file.yml"            
    neurons:
      - debug:
          message: "hello"
```

Where `my_stt_correction_file.yml` would looks like the following: 
```yml
- input: "test"
  output: "order"
```

### Use both stt-correction and stt-correction-file

You can use both flag stt-correction and stt-correction-file in a synapse. 
This can be useful to set a correction file used as global file, and override input with stt-correction.

Syntax:
```yml
signals:
  - order:
      text: "<sentence>"
      stt-correction-file: "<path to yaml file>"  
      stt-correction:
        - input: "<sentence>"
          output: "<replacing sentence>" 
```

For example, if you define a `stt-correction-file` with the content bellow:
```yml
- input: "bla"
  output: "this"
```

And a synapse like the following
```yml
- name: "stt-correction-test"
  signals:
    - order:
        text: "this is my order"
        stt-correction-file: "correction.yml"
        stt-correction:
          - input: "test"
            output: "order"
```

If you pronounce "bla is my test", both `stt-correction-file` and `stt-correction` will be used to fix the pronounced order, resulting "this is my order".

>**Note:** `stt-correction` has precedence over `stt-correction-file`. 
If an input is declared in `stt-correction` and in `stt-correction-file`, the output will be taken from the `stt-correction` option.

### Order with arguments
You can add one or more arguments to an order by adding bracket to the sentence.

Syntax:
```yml
signals:
    - order: "<sentence> {{ arg_name }}"
    - order: "<sentence> {{ arg_name }} <sentence>"
    - order: "<sentence> {{ arg_name }} <sentence> {{ arg_name }}"
```

Example:
```yml
signals:
    - order: "I want to listen {{ artist_name }}"
    - order: "start the {{ episode_number }} episode"
    - order: "give me the weather at {{ location }} for {{ date }}"
```

Here, an example order would be speaking out loud the order: "I want to listen Amy Winehouse"
In this example, both word "Amy" and "Winehouse" will be passed as an unique argument called `artist_name` to the neuron.

If you want to send more than one argument, you must split your argument with a word that Kalliope will use to recognise the start and the end of each arguments.
For example:  "give me the weather at {{ location }} for {{ date }}"
And the order would be: "give me the weather at Paris for tomorrow"
And so, it will work too with: "give me the weather at St-Pierre de Chartreuse for tomorrow"

See the **input values** section of the [neuron documentation](neurons) to know how to send arguments to a neuron.

>**Important note:** The following syntax cannot be used: "<sentence> {{ arg_name }} {{ arg_name2 }}" as Kalliope cannot know when a block starts and when it finishes.

## Notes

> **Important note:** SST engines can misunderstand what you say, or translate your sentence into text containing some spelling mistakes.
For example, if you say "Kalliope please do this", the SST engine can return "caliope please do this". So, to be sure that your speaking order will be correctly caught and executed, we recommend you to test your STT engine by using the [Kalliope GUI](kalliope_cli.md) and check the returned text for the given order.

> **Important note:** STT engines don't know the context. Sometime they will return an unexpected word.
For example, "the operation to perform is 2 minus 2" can return "two", "too", "to" or "2" in english.

> **Important note:** Kalliope will try to match the order in each synapse of its brain. So, if an order of one synapse is included in another order of another synapse, then both synapses tasks will be started by Kalliope.

> For example, you have "test my umbrella" in a synapse A and "test" in a synapse B. When you'll say "test my umbrella", both synapse A and B
will be started by Kalliope. So keep in mind that the best practice is to use really different sentences with more than one word for your order.