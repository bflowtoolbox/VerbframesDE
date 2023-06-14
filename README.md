# VerbframesDE - Syntactic Verb Frames and Synsets

This file describes the format of both files verbframes.json and synsets.json, a JSON representation of german lexical verb frames and synsets, resp. The dataset was built for use with Business Process Models (BPMs), especially to represent the main activity of a label. This means that it reflects only a limited scope of domains and has to be hugely extended to be useful in non-business setups.

## Verb Frames in JSON

We want to represent verbs with their syntactic slots in the German language. This means that we have to represent
- all four cases of the German language,
- slot labels describing German parts of speech with relation to verbs and verb phrases,
- slots with a specific word filling,
- slots with a specific relation to the verb, but with no fixed filling,
- slots with multiple options of parts of speach, but the same semantic relation to the verb,
- slots filled with a nested phrasal structure, mostly noun phrases.

Frequently used tagsets are structured best for tagging the English language and are not adequate for representing German phrases in a concise manner. So we extended the tagset used for verb frames in GermaNet, using their tags, starting with capital letters; Our additional tags start with lowercase letters.

We will start with a basic example, representing the German verb *bezahlen* – to pay:

> {"vfin":"bezahlen","optional":["NN","AN","DN"]}

*bezahlen* is the head of the verb phrase, labeled by the key *vfin*, meaning *finite verb form*. Our verb can be used with a subject (who pays), an accusative object (what is paid) and a dative object (who is paid), but these roles are by no means fixed, so we put the *NN*, *AN* and *DN* tags in the *optional* list.

> {"vfin":"machen","AN":"Gebrauch","von+D":"Möglichkeit","optional":["NN"]}

*machen* is the head of the verb phrase. The verb has a fixed prepositional complement, namely *von + Möglichkeit* (most often expressed as *von der Möglichkeit*), which is represented using the preposition *von* and a *D* for the dative case as a key and *Möglichkeit* as the value. Furthermore the slot of the accusative object is filled with the noun *Gebrauch*. When words fill slots, the value is the lemma of the word (nominative singular of a noun, infinitive of a verb or base form of an adjective).

We use the following tags:
| Tag | Description |
|-----|-------------|
| NN  | subject     |
| AN  | accusative object |
| DN  | dative object |
| GN  | genitive object |
| NG  | nominative complement |
| AG  | accusative complement |
| NE  | nominative *es*; value is always "es"|
| B   | adverb |
| AR  | reflexive accusative pronoun |
| DR  | reflexive dative pronoun |
| AI  | secondary verb     |
| *prep*+*case* | preposional phrase governed by the preposition *prep*, putting the noun phrase in the case *case* |
| n   | noun |
| det | determiner |
| adj | adjective |
| p   | perfect participle; used attributively like *adj*, but the underlying verb can be mapped to the main action of the label |
| head   | head of the phrase, see below |

In case of fixed fillings, the values are always the lemmas of the filling words, if not mentioned otherwise.
In more complex verb phrases one lemma is not sufficient to describe a fixed slot filling. Then instead of the lemma the value becomes a JSON object. This object contains keys according to the phrase to be specified, usually a noun phrase. The *head* key holds the head of the phrase, and for additional slots to describe noun phrases, the keys *n*, *det* and *adj* are used.

> {"vfin":"schenken","AN":{"adj":"kein","head":"Beachtung"},"optional":["NN","DN"]}

In this example for the verb phrase *jemandem keine Beachtung schenken* the accusative object had to be encoded as a JSON object, because the adjective "kein" determines the meaning substantially. The noun "Beachtung" fills the *AN* slot (due to the "head" key), and the adjective "kein" determines that noun inside the noun phrase.

> {"vfin":"entsprechen","DN":"Tatsache","optional":["NN"]}

Note from this example that only the uninflected lemma of the object is listed. This means that lemmatization has to be executed before matching the verbframes in order to deal with the phrase *den Tatsachen entsprechen*.

If the slots are not filled specifically, the tags can be part of the following 3 lists, depending on if the slots are filled if this verb frame matches:
| Tag list  | Description |
|-----------|-------------|
| optional  | can be filled |
| mandatory | must be filled, but no concrete lemma is required |
| forbidden | must not be filled; helps to distinguish verb frames for the same verb but having different syntactic slots |

Examples:

> {"vfin":"wenden","optional":["NN"],"mandatory":["AR","an+A"]}

> {"vfin":"abtreten","optional":["NN"],"forbidden":["AN","DN/an+A"]}

> {"vfin":"abtreten","optional":["NN","AN"],"mandatory":["DN/an+A"]}

expressing the phrases *sich an jmd. wenden* and *abtreten* = *to step back* (but *not* the phrase *etw. an jmd. abtreten* = *hand sth. over*, shown in the third row), resp.

Each verb frame can be part of synsets. The key *synsets* holds the list of the corresponding synset ids.

## Synsets in JSON

The file synsets.json contains a list of synset specifications. Each synset JSON object uses the following keys:
| Key     | Type | Description |
|---------|------|-------------|
| id      | number | the synset id |
| name    | string | a descriptive name |
| type    | string | one of "act", "be", "become" |
| subsets | list of synset objects | synsets that are deemed more specific than the current synset |

This simple structure allows us to represent trees of synsets. Synsets can be purely synthetic, i. e. have no member frames at all, they even can be just for grouping purposes, e. g. they can represent domains in which verb frames usually make sense.

Although the tree structure organizes synsets in a disjoint way, verb frames can still be grouped into multiple synsets, allowing overlap between the synsets.

As described in the article, synsets are marked as *actions*, *being in a state* and *transitioning to a state*, which is marked under the key *type*, using the values "act", "be" and "become", resp.