# compact Volumetric Acoustic Sensor (cVAS)

This book describes the compact Volumetric Acoustic Sensor.

In particular it collects the Python based Jupyter notebook pages developed to understand Jupyter based processing and to demonstrate the different steps that may be carried out for processing the data.

This book is a Work-In-Progress of cVAS processing.

A second part addresses cPAM processing.

Initially, notebook pages are added for processing, followed by more descriptive texts.

    jb build cVAS_book 
    jupyter nbconvert --to latex cVAS_book/*.ipynb --output-dir=cVAS_book/cVAS_TEX --template-file=style_mybw.tex.j2