\documentclass[11pt]{article}
\usepackage{geometry}
\geometry{margin=1in}
\usepackage{amsmath, amssymb}
\usepackage{listings}
\usepackage{graphicx}
\usepackage{hyperref}
\usepackage{titlesec}
\usepackage{enumitem}

\titleformat{\section}{\large\bfseries}{\thesection}{1em}{}
\titleformat{\subsection}{\normalsize\bfseries}{\thesubsection}{1em}{}
\titleformat{\subsubsection}[runin]{\bfseries}{\thesubsubsection}{1em}{}[.]

\title{\textbf{Probabilistic Knowledge Graph Reasoning with DistMult, Pyro, and Neo4j}}
\author{}
\date{}

\begin{document}

\maketitle

\section*{Overview}

This project implements a full pipeline for building, training, and reasoning over a probabilistic biomedical knowledge graph using the \textbf{PrimeKG} dataset. It combines \textbf{neural link prediction}, \textbf{Bayesian inference}, and \textbf{multi-hop probabilistic querying} with Neo4j for graph inspection and visualization.

\section*{Main Features}

\begin{itemize}[leftmargin=*]
  \item Knowledge Graph Embedding using the DistMult model.
  \item Triple-level confidence estimation via sigmoid-transformed scores.
  \item Bayesian modeling using a Beta-Bernoulli hierarchical prior (via Pyro).
  \item Posterior inference via MCMC sampling (NUTS).
  \item Optional Neo4j graph integration with prior/posterior scores.
  \item Multi-hop probabilistic reasoning over the knowledge graph.
\end{itemize}

\section*{Dependencies}

\textbf{Primary libraries used:}
\begin{itemize}[leftmargin=*]
  \item \texttt{pykeen} – for loading PrimeKG and training DistMult.
  \item \texttt{pyro-ppl} – for probabilistic modeling and inference.
  \item \texttt{torch-geometric} – backbone for DistMult.
  \item \texttt{neo4j} – for Cypher graph querying and upload.
  \item \texttt{pandas}, \texttt{numpy}, \texttt{matplotlib} – for data analysis and plotting.
\end{itemize}

\noindent Install with:
\begin{lstlisting}[language=bash]
pip install --upgrade pip setuptools wheel torch-geometric pykeen neo4j pyro-ppl
\end{lstlisting}

\section*{Workflow Summary}

\subsection*{1. Load PrimeKG}
We use PyKEEN to load the PrimeKG biomedical knowledge graph, splitting it into training, validation, and test sets.

\subsection*{2. Train DistMult Model}
A DistMult embedding model is trained using 100,000 sampled triples with margin-based negative sampling. The model is optimized using the Adam optimizer for approximately 100 epochs.

\subsection*{3. Compute Prior Probabilities}
We transform DistMult scores via the sigmoid function to yield probabilistic priors for each triple, representing the model's raw confidence.

\subsection*{4. Bayesian Posterior Inference}
A Beta-Bernoulli model is used to refine priors:
\begin{itemize}
    \item Each prior becomes a Beta distribution.
    \item Observed true triples are modeled with Bernoulli likelihood.
    \item Posterior inference is conducted using NUTS-based MCMC in Pyro.
\end{itemize}

\subsection*{5. Posterior Storage and Visualization}
Posterior mean probabilities are stored and optionally uploaded to a Neo4j database. These values are used as edge weights in graph queries.

\subsection*{6. Multi-hop Probabilistic Reasoning}
We build an in-memory graph using the posterior probabilities. The following utilities are defined:
\begin{itemize}
    \item \texttt{find\_paths()} – finds all valid paths up to $k$ hops.
    \item \texttt{aggregate\_path\_probs()} – combines multiple path probabilities using a noisy-or model.
    \item \texttt{query\_multihop()} – returns probabilistic multi-hop inference results and path traces.
\end{itemize}

\section*{Example Output}

\begin{verbatim}
P('de novo' AMP biosynthetic process → ADSL) ≈ 0.535 via 1 path(s)
  Path (1 hop): 'de novo' AMP biosynthetic process -[bioprocess_protein] → ADSL

P('de novo' AMP biosynthetic process → ATP metabolic process) ≈ 0.048 via 1 path(s)
  Path (5 hops): 
  'de novo' AMP biosynthetic process 
    → AMP biosynthetic process 
    → AMP metabolic process 
    → AMP phosphorylation 
    → ATP biosynthetic process 
    → ATP metabolic process
\end{verbatim}

As the number of hops increases, posterior probabilities decrease—demonstrating compounding uncertainty. This aligns with the principles of Bayesian inference and illustrates explainable, probabilistic path-based reasoning.

\section*{Applications}

\begin{itemize}[leftmargin=*]
  \item Biomedical hypothesis generation
  \item Drug target discovery via multi-hop reasoning
  \item Uncertainty-aware AI in knowledge graphs
  \item Transparent inference for biomedical graphs
\end{itemize}

\section*{Files Included}

\begin{itemize}[leftmargin=*]
  \item \texttt{Building\_a\_probabilistic\_KG\_DistMult\_Implementation.ipynb} – Main notebook
  \item \texttt{neo4j\_upload\_utils.py} – (Optional) Script for modular Neo4j upload
\end{itemize}

\section*{Running Instructions}

\begin{enumerate}[leftmargin=*]
  \item Open the notebook in Google Colab or a local Jupyter environment.
  \item Install dependencies using pip.
  \item Run all notebook cells in order.
  \item (Optional) Configure Neo4j credentials and enable data upload to a graph database.
\end{enumerate}

\section*{Acknowledgments}

\begin{itemize}[leftmargin=*]
  \item PyKEEN
  \item Pyro Probabilistic Programming Language
  \item PrimeKG Dataset
  \item Neo4j AuraDB (for cloud deployment)
\end{itemize}

\end{document}
