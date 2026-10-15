# NOFO Research Proposal Generator

A retrieval-augmented generation (RAG) workflow that reads an NIH funding notice (NOFO), finds the most relevant papers in a researcher's archive, brainstorms proposal ideas grounded in those papers, drafts a full proposal from a template, and scores it against NIH review criteria.

Built as the end-term/final project for the Johns Hopkins University *Applied Generative AI* program (AGAI).

## The scenario

A research lab has a large, loosely organized archive of publications. When a new NIH NOFO appears (here, an NIMH call on digital mental health), finding which past work fits, and turning it into a competitive proposal, is slow. This project automates that path.

## Pipeline

1. **Topic extraction.** An LLM reads the NOFO PDF and returns the funded topic: *"Optimize Digital Health Interventions For Mental Health."*
2. **Relevance retrieval.** All research PDFs are split into 9,106 chunks, embedded with `sentence-transformers/all-MiniLM-L6-v2`, and indexed in FAISS. A top-50 similarity search on the topic narrows the archive to 15 relevant papers.
3. **Summarization.** Each relevant paper is summarized by the LLM (with a token cap to stay inside the context window).
4. **Ideation.** The LLM proposes 5 ideas, each with a title, description, citation and source file path.
5. **Proposal drafting.** One idea is chosen (here, *Leveraging Social Determinants of Health to Inform Digital Mental Health Strategies*), and the LLM writes the full proposal using the source paper and a sample proposal as a structural template.
6. **LLM-as-judge evaluation.** The proposal is scored 1-5 on NIH criteria and returned as JSON.
7. **Human review.** The draft is meant to be edited by a person before submission.

## Evaluation results

| Criterion | Score |
|-----------|-------|
| Innovation | 4 / 5 |
| Significance | 5 / 5 |
| Approach | 3 / 5 |
| Investigator Expertise | 3 / 5 |

The judge flagged the *Approach* section as the weakest (needs more methodological detail) and asked for more specific examples of how social-determinants insights would be built into interventions.

## Tech stack

Python, LangChain, OpenAI `gpt-4o-mini` (temperature 0), Hugging Face sentence-transformers, FAISS, PyPDF, markdown-pdf.

## Getting started

1. Install dependencies: `pip install -r requirements.txt` (the notebook pins versions that work together; restart the kernel if prompted).
2. Copy `config.example.json` to `config.json` and add your OpenAI API key. This file is git-ignored.
3. Add your own inputs (see below).
4. Open `nofo_proposal_generator.ipynb` and run the cells in order.

## Inputs

The NOFO PDF, the research-paper archive and the sample proposal templates were supplied with the course and are **not included** here. To reuse the pipeline, place a NOFO PDF, a folder of research PDFs, and one or more sample proposals in the paths the notebook expects (or edit the paths at the top of each step).

## Known limitations

- Idea selection relies on a regex to pull the source file path out of the LLM's text; a formatting change in the output breaks it.
- The evaluator parses the model's reply as JSON by slicing off a code fence, which is brittle.
- Citations and file paths are not verified against the actual papers.
- The judge and generator are the same model, so scores are likely generous.

## Ideas for improvement

- A multi-agent workflow where separate agents draft, review and refine each proposal section.
- Automatic citation verification (authors, years, file paths).
- Structured output (JSON schema / function calling) instead of regex and string slicing.

## License

MIT
