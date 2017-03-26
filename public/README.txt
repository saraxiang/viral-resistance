Right now we're immediately loading html for article (template 0), and template0 explicitly uses (generically named) html elements from template 0.

numQuestions initialized by loadTemplate()
questionsCount initialized by loadTemplate() to be 0
questionsCount is incremented after new question is loaded

askQuestions is flipped to true by loadTemplate()
askQuestions is flipped to false by click catch if questionsCount = numQuestions (no more questions)

switchQuestions is flipped to false after new question is loaded 
switchQuestions is flipped to true by click catch

if (askQuestions)
