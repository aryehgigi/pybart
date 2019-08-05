# UD2UDE

This (badly-named) project includes couple of mini-projects, all related to the main goal of my thesis.
1. **Converter:** python(3.6) converter, aimed to replace core-nlp's Java converter, and with my researched add-ins.
2. **Model:** spaCy trained model based on UD (and PENN converted to UD) dataset.
3. **Demo:** JS and python code, making use of the converter.
4. **spacy2odin:** jsons converter for annotating data using the Model (above) for odin-son indexing.

# Converter
The converter converts UD (v1.4) to enhancedUD, enhancedUD++, and extra-enhancements (discovered as part of my thesis).
It supports Conll-U and Odin formats (and some conversions between them).

Generally, I tried to maintain the same behavior (mentioned [here](http://www.lrec-conf.org/proceedings/lrec2016/pdf/779_Paper.pdf), and that was implemented by [core-NLP java converter](https://nlp.stanford.edu/software/stanford-dependencies.shtml)) as much as reasonable.

The converter coveres the following conversions:

|                                                                                 | [paper](https://nlp.stanford.edu/pubs/schuster2016enhanced.pdf)   (or [here](http://www.lrec-conf.org/proceedings/lrec2016/pdf/779_Paper.pdf)) | [UD formal guidelines   (v2)](https://universaldependencies.org/u/overview/enhanced-syntax.html)          | coreNLP   code  | Converter       | notes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|-------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|-----------------|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| nmod/acl/advcl   case info                                                          | eUD                                                                                                                                            | eUD   (under 'obl' for v2)                                                                                | eUD             | eUD             | 1.   Even though multi-word prepositions are processed only under eUD++, it is   still handled under eUD to add it in the case information.<br>2. Lowercased (and not lemmatized - important for MWP)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Passive   agent                                                                     | -                                                                                                                                              | -                                                                                                         | eUD             | eUD             | Only   if the nmod both has a "by" son and has an 'auxpass' sibling, then   instead of nmod:by we fix to nmod:agent                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| conj   case info                                                                    | eUD                                                                                                                                            | eUD                                                                                                       | eUD             | eUD             | 1.   Adds the type of conjunction to all conjunct relations<br>2. Some multi-word coordination markers are collapsed to conj:and or   conj:negcc                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Process   Multi-word prepositions                                                   | eUD++                                                                                                                                          | eUD   (?)                                                                                                 | eUD++           | eUD++           | Predetermined   lists of 2w and 3w preps.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Demote   quantificational modifiers (A.K.A Partitives and light noun constructions) | eUD++                                                                                                                                          | (see   [here](https://universaldependencies.org/u/overview/enhanced-syntax.html#additional-enhancements)) | eUD++           | eUD++           | Predetermined   list of the quantifier or light noun.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Conjoined   prepositions and prepositional phrases                                  | eUD++                                                                                                                                          | -                                                                                                         | eUD++           | eUD++           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Propagated   governors and dependents                                               | eUD   (A, B, C)                                                                                                                                | eUD   (A, B, C, D)                                                                                        | eUD   (A, B, C) | eUD   (A, B, C) | 1.   This includes: (A) conjoined noun phrases, (B) conjoined adjectival phrases,   (C) subjects of conjoined verbs, and (D) objects of conjoined verbs.<br>2. Notice (D) is relevant to be added theoretically but was omitted for   practical uncertainty (see 4.2 at the paper).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Subjects   of controlled verbs                                                      | eUD                                                                                                                                            | eUD                                                                                                       | eUD             | eUD             | 1.   Includes the special case of 'to' with no following verb ("he decided   not to").<br>2. Heuristic for choosing the propagated subject (according to coreNLP   docu): if the control verb has an object it is propagated as the subject of   the controlled verb, otherwise they use the subject of the control verb.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Subjects   of controlled verbs - when 'to' marker is missing                        | ?                                                                                                                                              | ?                                                                                                         | -               | extra           | 1.   Example: "I started reading the book"<br>2. For some reason not included in the coreNLP code, unsure why                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Relative   pronouns                                                                 | eUD++                                                                                                                                          | eUD   (?)                                                                                                 | eUD++           | eUD++           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Reduced   relative clause                                                           | -                                                                                                                                              | eUD   (?)                                                                                                 | -               | extra           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Subjects   of adverbial clauses                                                     | -                                                                                                                                              | -                                                                                                         | -               | extra           | Heuristic   for choosing the propagated entity:<br>1. If the marker is "to", the object (if it is animated - but for   now we don’t enforce it) of the main clause is propagated as subject,   otherwise the subject of the main clause is propagated.<br>2. Else, if the marker is not one of "as/so/when/if" (this   includes no marker at all which is mostly equivalent to "while"   marker), both the subject and the object of the main clause are equivalent   options (unless no object found, then the subject is propagated).                                                                                                                                                                                                                                                                                     |
| Noun-modifying   participles                                                        | (see   [here](https://www.aclweb.org/anthology/W17-6507))                                                                                      | -                                                                                                         | -               | extra           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Correct   possible subject of Noun-modifying participles                            | -                                                                                                                                              | -                                                                                                         | -               | extra           | 1.   This is a correctness of the subject decision of the previous bullet.<br>2. If the noun being modified is an object/modifier of a verb with some   subject, then that subject might be the subject of the Noun-modifying   participle as well. (it is uncertain, and seems to be correct only for the   more abstract nouns, but that’s just a first impression).                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Propagated   modifiers (in conjunction constructions)                               | -                                                                                                                                              | -                                                                                                         | -               | extra           |       Heuristics and assumptions:<br>1. Modifiers that appear after both parts of the conjunction may (the ratio   should be researched) refer to both parts. Moreover, If the modifiers father   is not the immediate conjunction part, then all the conjunction parts between   the father and the modifier are (most probably) modified by the   modifier.<br>2. If the modifier father is the immediate conjunction part, we propagate   the modifier backward only if the new father, doesn't have any modifiers sons   (this is to restrict a bit the amount of false-positives).<br>3. We don’t propagate modifier forwardly (that is, if the conjunct part   appears after the modifier, we assume they don’t refer).<br>4. Should be tested for cost/effectiveness as it may bring many   false-positives.       |
| Locative   and temporal adverbial modifier propagation                              | -                                                                                                                                              | -                                                                                                         | -               | extra           | 1.   Rational: If a locative or temporal adverbial modifier is stretched away from   the verb through a subject/object/modifier(nmod) it should be applied as well   to the verb itself.<br>2. Example: "He was running around, in these woods here".                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Subject   propagation of 'dep'                                                      | -                                                                                                                                              | -                                                                                                         | -               | extra           | Rational:   'dep' is already problematic, as the parser didn't know what relation to   assign it.     In case the secondary clause doesn't have a subject, most probably it   should come from the main clause. It is probably an advcl/conj/parataxis/or   so that was missing some marker/cc/punctuation/etc.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ellipsis                                                                            | (see   [here](https://www.aclweb.org/anthology/W17-0416) and   [here](https://arxiv.org/pdf/1804.06922.pdf))                                   | eUD   (?) (suited only for v2 as handles orphans)                                                         | -               | TBD             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Apposition   propagation                                                            | (see   [here](https://arxiv.org/pdf/1603.01648.pdf))                                                                                           | -                                                                                                         | -               | TBD             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| nmod   propagation                                                                  | -                                                                                                                                              | -                                                                                                         | -               | TBD             | 1.   Through conj is already covered<br>2. Through advmod is promising<br>3. Through subject/object/-nmod-chain - still under research                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Active-passive   alteration                                                         | (see   [here](https://www.aclweb.org/anthology/W17-6507))                                                                                      | -                                                                                                         | -               | TBD             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Copula   alteration                                                                 | -                                                                                                                                              | -                                                                                                         | -               | TBD             | Add   a verb placeholder, reconstruct the tree as if the verb was there.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
