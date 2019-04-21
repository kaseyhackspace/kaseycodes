---
title: The one about N-Grams and Markov Chains
date: 2019-04-21 08:30:00
tags: 
    - python
    - numpy
    - n-grams
    - nlp
    - markov chains
---
Today we are going to make a Markov Chain to generate text from a training dataset we are going to feed it. So how do we start? We start by first implementing n-grams. N-grams are a list of n words that share a common border in a given string. It may sound confusing, so it is better that I implement it first then show by example:


```python
class NGram:

    def ngram(self, text, n=2):
        text_list = text.lower().split()
        grams = [tuple(text_list[index:index+n]) for index in range(0, len(text_list) - (n-1))]
        return grams

    def bigram(self, text):
        return self.ngram(text, n=2)

    def trigram(self, text):
        return self.ngram(text, n=3)
```

I defined a class called `NGram` that contains three functions. The first function `ngram` takes in a string variable `text` and an integer `n` to create an n-gram. By default, if no `n` is passed, then `n=2` and we get what's called a bigram (bi meaning 2). In line 4 I do some simple preprocessing of the text by converting everything to lowercase with the `lower` function and then splitting the string per word into a list with the `split` function (if you don't specify a separator in the `split` function, it splits the string by whitespace). n-grams are actually generated in line 5 where I generate a list of `tuples` using list indexing and list comprehension techniques.

`bigram` and `trigram` functions are used to make n-grams where `n=2` and `n=3` respectively. As you can see, they use the `ngram` function under the hood. Lets have a few examples so you can see what the piece of code does on out sample text: "Hello word, this is a sample text for consumption of the NGram class!"


```python
sample = "Hello word, this is a sample text for consumption of the NGram class!"
ngram = NGram()
# bi-gram example
print(ngram.bigram(sample))
```

    [('hello', 'word,'), ('word,', 'this'), ('this', 'is'), ('is', 'a'), ('a', 'sample'), ('sample', 'text'), ('text', 'for'), ('for', 'consumption'), ('consumption', 'of'), ('of', 'the'), ('the', 'ngram'), ('ngram', 'class!')]



```python
# tri-gram example
print(ngram.trigram(sample))
```

    [('hello', 'word,', 'this'), ('word,', 'this', 'is'), ('this', 'is', 'a'), ('is', 'a', 'sample'), ('a', 'sample', 'text'), ('sample', 'text', 'for'), ('text', 'for', 'consumption'), ('for', 'consumption', 'of'), ('consumption', 'of', 'the'), ('of', 'the', 'ngram'), ('the', 'ngram', 'class!')]



```python
# n-gram example where n=4
print(ngram.ngram(sample,n=4))
```

    [('hello', 'word,', 'this', 'is'), ('word,', 'this', 'is', 'a'), ('this', 'is', 'a', 'sample'), ('is', 'a', 'sample', 'text'), ('a', 'sample', 'text', 'for'), ('sample', 'text', 'for', 'consumption'), ('text', 'for', 'consumption', 'of'), ('for', 'consumption', 'of', 'the'), ('consumption', 'of', 'the', 'ngram'), ('of', 'the', 'ngram', 'class!')]


I do hope the examples above made n-grams clearer than I could explain them :). Now that we've implemented our NGram class, we can start building our shiny Markov Chain! The markov chain class is a bit lengthy, so I chose to write docstrings under the function names instead so you have have a high level view of the class and dig in to investigate the code if you want.


```python
import numpy as np

class MarkovChain:
    
    def __init__(self):
        """
        Initialization creates empty dictionary
        """
        self.model = {}
    
    def calculate_probabilities(self):
        """
        per word in model, calculate the probability of next words following the current word
        based on the bigram pair and how many times the bigram pair appears in the training dataset
        """
        for key in self.model:
            total_sum = np.sum(self.model[key]['counts'])
            self.model[key]['probability'] = [x/total_sum for x in self.model[key]['counts']]
            
    def train_model(self,text):
        """
        The train_model method first generates bigrams from the given text string. The method then
        traverses each bigram and determines if the first word in the bigram already exists in the model.
        The behavior of the method per bigram is determined with the following cases:
        
        If the first element has not been added yet(does not have a key in the dictionary), the first 
        element is added to the dictionary with the second element added in the `values` list with a 
        corresponding occurance count of 1.
        
        If the first element has already been added but the second element has not yet been added to the
        `values` list, it appends the second element to the `values` list with a corresponding occurance count 
        of 1. 
        
        If both the first and second elements of the bigram already exist, just increment the number of
        occurances of the second element by 1.
        
        After all bigrams have been processed, the train_model method calls the calculate_probability method
        to calculate the probability of choosing the next word randomly given the current work in the Markov
        Chain.
        
        The final model would have the following structure:
        
        {
            'word-1': {
                values: [<list_of_probable_words>]
                counts: [<number_of_occurences_per_word_in_values>]
                probability: [<probability_of_occurence_per_word_in_values>]
            },
            'word-2': {
                values: [<list_of_probable_words>]
                counts: [<number_of_occurences_per_word_in_values>]
                probability: [<probability_of_occurence_per_word_in_values>]
            },
            ...,
            'word-n': {
                values: [<list_of_probable_words>]
                counts: [<number_of_occurences_per_word_in_values>]
                probability: [<probability_of_occurence_per_word_in_values>]
            }
        }
        """
        ngram = NGram()
        
        bigrams = ngram.bigram(text)
        
        for bigram in bigrams:
            if bigram[0] not in self.model:
                self.model[bigram[0]]= {
                    'values': [bigram[1]],
                    'counts': [1],
                    'probability': []
                }
            else:
                if bigram[1] not in self.model[bigram[0]]['values']:
                    self.model[bigram[0]]['values'].append(bigram[1])
                    self.model[bigram[0]]['counts'].append(1)
                else:
                    index = self.model[bigram[0]]['values'].index(bigram[1])
                    self.model[bigram[0]]['counts'][index] = self.model[bigram[0]]['counts'][index] + 1
            
            self.calculate_probabilities()
    
    def generate_text(self, n):
        """
        This method generates a random sentence of n words. It selects a random starting point from the model
        then chooses a random hop to the next word based on the probability array of the word list n times.
        
        Note that it is possible to have a word that does not have next hops, we gracefully handle that with
        a try except case that returns the sentence prematurely if ever that happens.
        """
        sentence = ''
        current_word = np.random.choice(list(self.model.keys()))
        sentence += current_word
        for index in range(n):
            try:
                current_word = np.random.choice(self.model[current_word]['values'],p=self.model[current_word]['probability'])
                sentence += ' '+current_word
            except:
                return sentence   
        return sentence
    
    def generate_sentence(self):
        """
        This method generates a random sentence of . It selects a random starting point from the model
        then chooses a random hop to the next word based on the probability array of the word list until
        it encounters a ".".
        
        Note that it is possible to have a word that does not have next hops, we gracefully handle that with
        a try except case that returns the sentence prematurely if ever that happens.
        """
        sentence = ''
        current_word = np.random.choice(list(self.model.keys()))
        sentence += current_word
        while '.' not in current_word:
            try:
                current_word = np.random.choice(self.model[current_word]['values'],p=self.model[current_word]['probability'])
                sentence += ' '+current_word
            except:
                return sentence   
        return sentence
    
```

Now that we have the Markov Chain class implemented, we can test out the code! For the training dataset, I chose wikipedia's summary of the [Avengers: infinity war](https://en.wikipedia.org/wiki/Avengers:_Infinity_War#Plot) movie.


```python

training_text = "Having acquired the Power Stone, one of the six Infinity Stones, from the planet Xandar, Thanos and his lieutenants—Ebony Maw, Cull Obsidian, Proxima Midnight, and Corvus Glaive—intercept the spaceship carrying the surviving Asgardians. As they extract the Space Stone from the Tesseract, Thanos subdues Thor, overpowers Hulk, and kills Loki. Heimdall sends Hulk to Earth using the Bifröst before being killed. Thanos departs with his lieutenants and obliterates the ship. Hulk crash-lands at the Sanctum Sanctorum in New York City, reverting to Bruce Banner. He warns Stephen Strange and Wong about Thanos' plan to kill half of all life in the universe; in response, Strange recruits Tony Stark. Maw and Obsidian arrive to retrieve the Time Stone from Strange, drawing the attention of Peter Parker. Maw captures Strange, but fails to take the Time Stone due to an enchantment. Stark and Parker pursue Maw's spaceship, Banner contacts Steve Rogers, and Wong stays behind to guard the Sanctum. In Edinburgh, Midnight and Glaive ambush Wanda Maximoff and Vision in order to retrieve the Mind Stone in Vision's forehead. Rogers, Natasha Romanoff, and Sam Wilson rescue them and take shelter with James Rhodes and Banner at the Avengers Facility. Vision offers to sacrifice himself by having Maximoff destroy the Mind Stone to keep Thanos from retrieving it. Rogers suggests they travel to Wakanda, which he believes has the resources to remove the stone without destroying Vision. The Guardians of the Galaxy respond to a distress call from the Asgardian ship and rescue Thor, who surmises that Thanos seeks the Reality Stone, which is in the possession of the Collector on Knowhere. Rocket and Groot accompany Thor to Nidavellir, where they and Eitri create Stormbreaker, a battle-axe capable of killing Thanos. On Knowhere, Peter Quill, Gamora, Drax, and Mantis find Thanos with the Reality Stone already in his possession. Thanos kidnaps Gamora, his adopted daughter, who reveals the location of the Soul Stone to save her captive adopted sister Nebula from torture. Thanos and Gamora travel to Vormir, where Red Skull, keeper of the Soul Stone, informs him the stone can only be retrieved by sacrificing someone he loves. Thanos reluctantly kills Gamora, earning the stone. Nebula escapes captivity and asks the remaining Guardians to meet her on Thanos' destroyed homeworld, Titan. Stark and Parker kill Maw and rescue Strange. Landing on Titan, they meet Quill, Drax, and Mantis. The group forms a plan to remove Thanos' Infinity Gauntlet after Strange uses the Time Stone to view millions of possible futures, seeing only one in which Thanos loses. Thanos arrives, justifying his plans as necessary to ensure the survival of a universe threatened by overpopulation. The group subdues him until Nebula deduces that Thanos has killed Gamora. Enraged, Quill attacks him, allowing Thanos to break the group's hold and overpower them. Stark is seriously wounded by Thanos, but is spared after Strange surrenders the Time Stone to Thanos. In Wakanda, Rogers reunites with Bucky Barnes before Thanos' army invades. The Avengers, alongside T'Challa and the Wakandan forces, mount a defense while Shuri works to extract the Mind Stone from Vision. Banner, unable to transform into the Hulk, fights in Stark's Hulkbuster armor. Thor, Rocket, and Groot arrive to reinforce the Avengers; Midnight, Obsidian, and Glaive are killed and their army is routed. Thanos arrives and despite Maximoff's attempt to destroy the Mind Stone, removes it from Vision's head, killing him.Thor severely wounds Thanos, but Thanos activates the completed Infinity Gauntlet and teleports away. Half of all life across the universe disintegrates, including Barnes, T'Challa, Groot, Maximoff, Wilson, Mantis, Drax, Quill, Strange, and Parker, as well as Maria Hill and Nick Fury, although Fury is able to transmit a signal to Carol Danvers first. Stark and Nebula remain on Titan while Banner, M'Baku, Okoye, Rhodes, Rocket, Rogers, Romanoff, and Thor are left on the Wakandan battlefield. Meanwhile, Thanos watches a sunrise on another planet."
```

In the code below I try splitting the model up first by sentence then run a `for` loop to train the model per sentence.


```python
chain = MarkovChain()
for sentence in training_text.split('.'):
    chain.train_model(sentence)
```

Let's see what the model comes up with when we generate 10 sentences that are 20 words long:


```python
for index in range(1,11):
    print(str(index)+'. '+chain.generate_text(20))
```

    1. wanda maximoff and sam wilson rescue strange recruits tony stark is able to thanos to earth using the sanctum sanctorum in
    2. midnight, and wong stays behind to earth using the soul stone, one of the universe disintegrates, including barnes, t'challa, groot, maximoff,
    3. rescue them and despite maximoff's attempt to view millions of the tesseract, thanos departs with james rhodes and mantis find thanos
    4. knowhere, peter quill, gamora, his lieutenants—ebony maw, cull obsidian, proxima midnight, obsidian, and parker, as they travel to reinforce the remaining
    5. which is seriously wounded by overpopulation
    6. distress call from vision's forehead
    7. group's hold and teleports away
    8. fury is seriously wounded by thanos, but fails to an enchantment
    9. acquired the possession of the collector on thanos' destroyed homeworld, titan while banner, m'baku, okoye, rhodes, rocket, rogers, and rescue strange
    10. remove the time stone without destroying vision in new york city, reverting to thanos and groot arrive to guard the ship


Now let's try training the model by feeding it the whole `training_text` corpus instead and generate sentences that end in a period (.):


```python
chain = MarkovChain()
chain.train_model(training_text)    
for index in range(1,11):
    print(str(index)+'. '+chain.generate_sentence())
```

    1. universe; in stark's hulkbuster armor.
    2. be retrieved by sacrificing someone he believes has killed and despite maximoff's attempt to remove thanos' plan to transform into the time stone to earth using the group forms a distress call from strange, drawing the wakandan battlefield.
    3. midnight, and parker kill half of killing thanos.
    4. all life in vision's forehead.
    5. survival of killing him.thor
    6. but thanos seeks the asgardian ship and nick fury, although fury is seriously wounded by overpopulation.
    7. thor to save her on titan, they and asks the sanctum.
    8. distress call from strange, drawing the planet xandar, thanos arrives and gamora travel to earth using the possession of peter parker.
    9. wilson rescue strange.
    10. knowhere.


Hopefully you found the sentences generated by the Markov Chain amusing :D. For more information about n-grams and Markov Chain, just do a regular Google search and that would lead you to the right direction!

