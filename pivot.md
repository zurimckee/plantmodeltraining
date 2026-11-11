# potential pivot
## diagnostic model -> deterministic diagnostic
**since we are running low on time and can't find a viable dataset to train the plant disease diagnoser on, it may be a better idea to pivot to an entirely/mostly deterministic approach that leans on the classifier model that's already almost done**

- the classifier model can't output characteristics or features that it sees 

## possible deterministic design choices
- user picks the symptoms after the classifier model identifies the plant species
    - the app asks “What do you see?” with checkboxes (yellow leaves, brown tips, spots, wilting, drooping, pests). A table maps species + symptom to likely causes and fixes.
- simple image heuristic
    - measure how much of the leaf area is yellow or brown (color threshold check) - then preselect "possible yellowing" for the suer to select
    - quick but fragile
        - bg, lighting, flowers could fool it
- zero shot vision language model
    - prompt it with phrases
    - needs no training data but is still an ai component
    - accuracy on plant symptoms is unproven


## how the pivot (leaning towards bullet 1) would change our deliverable
- rewrite the evaluation measure in the next report instead of quietly dropping it. For example: advice-table correctness reviewed against cited sources, plus the species model’s accuracy.
- The advice has to come from somewhere credible
    - university extension services
    - zenodo care table 
    - cite whatever you use 
    - "general guidance not professional diagnosis" app in the note
- species model remains main ml contribution 
    - improving top 1 accuracy matters even more now but we can devote more energy towards it since the disease identifier is deterministic/non-issue