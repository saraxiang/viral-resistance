A person’s view in the game is defined by 
	state array (head of array is current state, remaining elements are future states - can grow and change based on user decisions in current state)
	currentState (contains info specific to current state)
	values of global variables 

MUST BE MAINTAINED IN DATABASE
	if (state object &&have type: inner) musthave article: INT
	when transitioning to new state, always set loadedAssets = false 
	after loadedAssets is set to true, variables like switchQuestion, loadedArticle need to be defined in currentState, and will
		be cased on (so they must exist!!!)












Right now we're immediately loading html for article (template 0), and template0() explicitly uses (generically named) html elements from template 0.

numQuestions initialized by loadTemplate()
questionsCount initialized by loadTemplate() to be 0
questionsCount is incremented after new question is loaded

askQuestions is flipped to true by loadTemplate()
askQuestions is flipped to false by click catch if questionsCount = numQuestions (no more questions)

switchQuestions is flipped to false after new question is loaded 
switchQuestions is flipped to true by click catch

	simplify all of the above to:

	numQuestions = databaseVal by loadState
	currentQuestion = 0 by loadState

	currentQuestion incremented after new question loaded

	if state type is inner and (currentQuestion < numQuestions) 

currentState is a dictionary that maintains values relevant to the current state type; they are overwritten after a 
state change but can be saved to datbase as necessary



