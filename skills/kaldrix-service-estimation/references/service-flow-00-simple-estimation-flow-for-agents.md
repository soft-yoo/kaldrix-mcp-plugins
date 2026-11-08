# Simple Estimation Flow for Agents

The estimation module deliberately starts with a small company catalog and frozen project estimates. A service can be
added with a manual quantity or with scopes taken from the BIM model, and every quantity is frozen in the estimate row.
Do not expect supplier marketplace integration or assignment groups. Estimate-level markup rules exist (see
`kaldrix://estimation/markup-rules`). `generate_estimate_pdf` prints the estimate PDF, the same file as the PDF button
in the Kaldrix application (see "Estimate PDF" below).
