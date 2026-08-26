Tokenizers:

  They serve one purpose: to translate text into data that can be processed by the model.
  wich means to create a token.

  A token is essentially a small unit of text that a model can understand.

  LLMs don’t read sentences the way humans do. Instead, they rely on tokens to process information.
  This allows them to handle complex sentences by breaking them into smaller, more manageable pieces.

  Models can only process numbers, so tokenizers need to convert our text inputs to numerical data.

  there are types of tokenisers :

     - Word-based
     - Character-based
     -  Subword-based

      1 - Word-based: split the raw text into words
         It splits a piece of text into words based on a delimiter. The most commonly used delimiter is space.

         Word-based tokenization can be easily done useing Python’s split() method.

         Example:

          “Is it weird I don’t like coffee?”

          By performing word-based tokenization with space as a delimiter, we get:

          [“Is”, “it”, “weird”, “I”, “don’t”, “like”, “coffee?”]


      2 - Character-based: split the raw text into individual characters
          This has two primary benefits:

            - The vocabulary is much smaller.
            - There are much fewer out-of-vocabulary (unknown) tokens, since every word can be built from characters.

          The logic behind this tokenization is that a language has many different words but has a fixed number of characters.

          Since the representation is now based on characters rather than words, one could argue that, intuitively, it’s less meaningful: each character doesn’t mean a lot on its own, whereas that is the case with words. 

      3 - Subword-based: Split the rare words into smaller meaningful subwords.
        The subword-based tokenization algorithms uses the following principles.

          -Do not split the frequently used words into smaller subwords.
          -Split the rare words into smaller meaningful subwords.

        For example, “boy” should not be split but “boys” should be split into “boy” and “s”.
        This will help the model learn that the word “boys” is formed using the word “boy” with slightly different meanings but the same root word.

        note that both boy and s are tokens.

        Most models which have obtained state-of-the-art results in the English language use some kind of subword-tokenization algorithms.



logits and probabilities :

  logits:
    Logits are the raw,  unbounded numerical scores a machine learning model produces before those scores are converted into probabilities.

    They can range from negative infinity to positive infinity.

    These raw numbers are useful for ranking (highest wins), but they are terrible for understanding confidence or probability.

    when dealing with only logits You cannot look at a raw logit number (like 5.0) and say "that means high confidence."
    The number itself is meaningless.
    Only the differences between the numbers matter.


  probabilities :

    Logits give a model an unconstrained output space during training and Probabilities must stay between 0 and 1 and, in a multi-class setting, must sum to 1.

    Models usually compute raw scores first, then apply the appropriate probability function (like softmax) when a probability is needed.

    so think of probabilities as the Clean, friendly numbers between 0 and 1 that add up to 1.00 (100%).
    These tell you the model's "confidence" in each answer.


  Example:

    model gives raw scores (logits) for Cat, Dog, Bird:
      Cat: 2.0    Dog: 1.0    Bird: 0.1

    These are just "points" – they rank answers (Cat wins), but they DON'T add up to 100% and can't be read as confidence.

    functions like Softmax or sigmoid turns those raw logits into real probabilities:
      Cat: 66%    Dog: 24%    Bird: 10%   (all add to 100%)


  TRAP: Don't stare at logit numbers alone. A score of 5.0 doesn't mean "more confident" than 2.0.
  Confidence comes from the GAPS between the scores.
  Big gaps = very sure. Small gaps = guessing.



autoregressive text generation:
  Autoregressive = Auto (self) + Regressive (predicting from past values)

    Auto means self. The model uses its own previous outputs as the next input.

    Regressive means predicting a value from past values. In simple words, we look at history, and we predict the next number.

    A simple example: predicting tomorrow's temperature from the temperatures of the last 7 days.

  So, putting them together:
    Autoregressive Model = A model that predicts the next token from all the previous tokens, one step at a time.






resouerses:
  https://medium.com/data-science/word-subword-and-character-based-tokenization-know-the-difference-ea0976b64e17
  https://huggingface.co/learn/llm-course/en/chapter2/4
  https://mehmetozkaya.medium.com/what-is-token-and-tokenization-62fafc30636e
  https://illuri-sandeep5454.medium.com/logits-vs-probabilities-understanding-neural-network-outputs-clearly-0e86a4256a0e
  https://telnyx.com/learn-ai/logits-ai
  https://outcomeschool.com/blog/autoregressive-models
  https://sebastianraschka.com/faq/docs/autoregressive-text-generation.html
  https://mbrenndoerfer.com/writing/autoregressive-generation-gpt-text-generation





youtube video i found of food that i want to try:
https://youtube.com/shorts/OZ13QXwvzBg?si=iewyNqHuQ6TS88nC

