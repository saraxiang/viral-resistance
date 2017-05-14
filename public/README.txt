A person’s view in the game is defined by 
	state array (head of array is current state, remaining elements are future states - can grow and change based on user decisions in current state)
	currentState (contains info specific to current state)
	values of global variables 

MUST BE MAINTAINED IN DATABASE
	if (state object && have type: inner or type complexInner) musthave article: INT
	when transitioning to new state, always set loadedAssets = false 
	after loadedAssets is set to true, variables like switchQuestion, loadedArticle need to be defined in currentState, and will
		be cased on (so they must exist!!!)

complexInnerState:
	article load same as innerState

	default: askQuestions: true, switchQuestions: false

	when link clicked: askQuestions: still true, switchQuestions: true, initialize CINodes array
	
	when choice clicked: flip switchQuestions to true, use the text of the choice to index into next CINode, add to beginning of CINodes array; UNLESS if a choice is "Go Back" or "On Second Thought" then pop off the first element of the CINodes array
	Note that here we can test for value of the node, not whether the name is "On Second Thought" or "Go back"

	when both askQuestions and switchQuestions are true (happens after link clicked, after choice clicked) == based on CINode, update form, flip switchQuestions to false; TEST IF AT LEAF NODE: clear CINodes array to empty, hide form HTML


Notes:
Right now outerImage is a global variable; in the future any images should be contained in currentState






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



********************** Form that can be used for inner loop ********************
<div id="form" class="radio-tile-group"></div>

.radio-tile-group {
  display: flex;
  flex-direction: column;
  width: 200px; 
}
.radio-tile-group .input-container {
  position: relative;
  margin: 0.5rem; 
}
.radio-tile-group .input-container .radio-button {
  opacity: 0;
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
  margin: 0;
  cursor: pointer; 
}
.radio-tile-group .input-container .radio-tile {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  border: 2px solid #079ad9;
  border-radius: 5px;
  padding: 1rem;
  transition: transform 300ms ease; 
}
.radio-tile-group .input-container .radio-tile-label {
  text-align: center;
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 1px;
  color: #079ad9; 
}
.radio-tile-group .input-container .radio-button:checked + .radio-tile {
  background-color: #079ad9;
  border: 2px solid #079ad9;
  color: white;
  transform: scale(1.1, 1.1); 
}
.radio-tile-group .input-container .radio-button:checked + .radio-tile .radio-tile-label {
  color: white;
  background-color: #079ad9; 
}



