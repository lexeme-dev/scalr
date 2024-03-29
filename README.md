# SCALR: Supreme Court Assessment of Legal Reasoning

The [SCALR benchmark](https://github.com/lexeme-dev/scalr) is a collection of 571 multiple choice questions designed to assess the legal reasoning
and reading comprehension ability of large language models. It is intended to measure
understanding of legal language, rather than the ability to memorize specific legal knowledge. The dataset will be made
publicly available as soon as data leakage concerns are resolved. In the meantime, it is available on request by email.

Each question gives the question presented for review in a particular Supreme Court case. The solver must
then determine which "holding" statement best corresponds to the question (i.e. which "holding" statement
is a description of the Court's holding in the case for which the question was presented). Here's an example:

> Question: Whether the Federal Arbitration Act preempts States from conditioning the enforcement of an arbitration agreement on the availability of particular procedures--here, class
-wide arbitration--when those procedures are not necessary to ensure that the parties to the arbitration agreement are able to vindicate their claims.
>
>A: holding that when the parties in court proceedings include claims that are subject to an arbitration agreement, the FAA requires that agreement to be enforced even if
a state statute or common-law rule would otherwise exclude that claim from arbitration
>
>B: holding that the Arbitration Act "leaves no place for the exercise of discretion by a district court, but instead mandates that district courts shall direct the partie
s to proceed to arbitration on issues as to which an arbitration agreement has been signed"
>
>C: holding that class arbitration "changes the nature of arbitration to such a degree that it cannot be presumed the parties consented to it by simply agreeing to submit
their disputes to an arbitrator"
>
>**D: holding that a California law requiring classwide arbitration was preempted by the FAA because it "stands as an obstacle to the accomplishment and execution of the full purposes and objectives of Congress," which is to promote arbitration and enforce arbitration agreements according to their terms**
>
>E: holding that under the Federal Arbitration Act, a challenge to an arbitration provision is for the courts to decide, while a challenge to an entire contract which includes an arbitration provision is an issue for the arbitrator

(*AT&T Mobility LLC v. Concepcion*, 563 U.S. 333 (2011))

Because questions presented for review in Supreme Court cases are not easily available prior to 2001, the
benchmark is limited to questions from cases decided in the 2001 Term and later.

## Construction

The data used to create this task comes from the following sources:
- Questions presented were gathered from the Supreme Court of the United States' website, which hosts
   PDFs of questions granted for review in each case dating back to the 2001 Term.
- Holding statements that comprise the "choices" for each question were compiled from (a)
   [CourtListener's collection](https://free.law/2022/03/17/summarizing-important-cases/)
   of parenthetical descriptions and (b)
   extraction of parenthetical descriptions from Courtlistener's and the [Caselaw Access Project](https://case.law/)'s collections of court decisions
   using [Eyecite](https://github.com/freelawproject/eyecite).

To ensure that "holding" statements would address the particular question presented, we limited
the set of cases to those in which exactly one question was granted for review. We also perform
some manual curation to exclude questions which are not answerable without specific knowledge of a case, e.g.:
>Whether this Court's decision in Harris v. United States, 536 U.S. 545 (2002), should be overruled.

### Choice Selection

To create choices for each question presented, we first filter our set of parenthetical descriptions
as follows:
- We limited our parenthetical descriptions to only those that begin with "holding that...",
as these are most  likely to describe the core holding of the case, rather than some peripheral issue.
- We use only parentheticals that describe Supreme Court cases. This avoids the creation
of impossible questions that ask the solver to distinguish between "holding" statements dealing with
exactly the same issue at different stages of appellate review.
- We then select for each case the longest parenthetical meeting the above criteria. We use the longest
  parenthetical because it is most likely to be descriptive enough to make the question answerable.


We then create a task for each case which has both a question presented and a "holding" statement
meeting the above requirements.
(While question-correct holding pairs are only for cases decided after 2001, we allow the use of
parentheticals describing any Supreme Court case as alternative answer choices.)
We then need to select the four alternative answer choices for each question in a manner that
makes the task challenging. To select choices that are at least facially plausible, we find
the four "holding" statements from the remaining pool that are most TF-IDF similar to the
question presented.

## Results

Our test set is composed of 30% of the full dataset (172 questions), chosen at random.
We report the performance of a number of different solvers below:

| Solver                                   | % Correct |
|------------------------------------------|-----------|
| Human*                                   | 86.0      |
| gpt-4-0314†                              | 81.4      |
| gpt-3.5-turbo-0301†                      | 64.0      |
| all-mpnet-base-v2 (CaseHOLD fine-tuned)‡ | 50.0      |
| all-mpnet-base-v2‡                       | 46.5      |
| all-distilroberta-v1‡                    | 44.2      |
| TF-IDF Bag-of-Words                      | 27.9      |

*Performance measured for a subset of 50 questions of the test set. Performance of models on this subset
is within 2% of the reported performance on the full test set.

†One-shot prompting with temperature 0.0.

‡Results for these models are computed via embedding dot-product similarity:
the choice with an embedding most similar to the question embedding is selected.
These models can be found on [Hugging Face](https://huggingface.co/sentence-transformers).

## Acknowledgements

SCALR was created by Faiz Surani and Varun Iyer. We thank Joshua Arp and other members of
Moot Court at UCSB for their assistance in evaluating the benchmark, and the Free Law Project
and Caselaw Access Project for their work in making legal data more accessible.

SCALR is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Any questions or comments regarding the dataset can be directed to Faiz Surani at
`faiz [at] faizsurani.com`.

SCALR was released as part of LegalBench, a collaborative effort to measure legal reasoning in large language models. Please cite LegalBench as follows:
```
@misc{guha2023legalbench,
      title={LegalBench: A Collaboratively Built Benchmark for Measuring Legal Reasoning in Large Language Models}, 
      author={Neel Guha and Julian Nyarko and Daniel E. Ho and Christopher Ré and Adam Chilton and Aditya Narayana and Alex Chohlas-Wood and Austin Peters and Brandon Waldon and Daniel N. Rockmore and Diego Zambrano and Dmitry Talisman and Enam Hoque and Faiz Surani and Frank Fagan and Galit Sarfaty and Gregory M. Dickinson and Haggai Porat and Jason Hegland and Jessica Wu and Joe Nudell and Joel Niklaus and John Nay and Jonathan H. Choi and Kevin Tobia and Margaret Hagan and Megan Ma and Michael Livermore and Nikon Rasumov-Rahe and Nils Holzenberger and Noam Kolt and Peter Henderson and Sean Rehaag and Sharad Goel and Shang Gao and Spencer Williams and Sunny Gandhi and Tom Zur and Varun Iyer and Zehua Li},
      year={2023},
      eprint={2308.11462},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```
