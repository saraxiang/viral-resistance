Some currently unused code (that can potentially be used in the future) and some stuff that needs to get done (additional things are noted as comments in game.html): 

********************** Future Things to Get Done *******************************
-> Handle case where sign in fails
-> Implement evidence and leads (global variables that can be modified within inner or outer states)
		var evidence; //defined in initApp, either pulled from database or set to [] (for new players)
		var leads;	//defined in initApp, either pulled from database or set to [] (for new players)
-> Implement a global "userHistory" variable to hold history for each completed state

********************** Outer Loop **********************************************

// This will be defined by handleOuter() and destroyed by nextStateAfterOuter
var outerImage;

// Load the asset inside Phaser's preload function
game.load.image('outerImage', 'assets/outer.jpg');

//This should be placed within Phaser's "update" function
if (currentStateType == 'outer') {
	console.log("In Outer Loop");
	handleOuter();
}

function handleOuter() {
	if (currentState['loadedAssets'] == false) {
		outerImage = game.add.sprite(200, 200, 'outerImage');

		var nextButton = game.add.button(0, 50, 'nextButton', nextStateAfterOuter, this);
		nextButton.scale.setTo(.2, .2);

		updateCurrentState('loadedAssets', true);
	}
}

function nextStateAfterOuter() {
	//delete...
	outerImage.destroy();
	//update stateArray(remove element at index 0)
	stateArray.splice(0,1);
	updateStateArray(stateArray);
	//clear current state and set loadedAssets to false
	updateCurrentState('loadedAssets', false);
}

******************* Form that can be used for an inner loop ********************
This is a form that displays questions one after the other on the screen, but handles the branching of questions based on previous responses
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

In function to handle user form choice selection, handle the case that user wants to "go back"
// TODO: don't just check for content to go back or not
if (CINodes[0]['choices'][id]['text'] == 'Go back' || CINodes[0]['choices'][id]['text'] == 'On second thought...') CINodes.splice(0,1);

Inside a function to generate the form
//each option in the form needs to pass its text or some other identifying info
to complexInnerStateHandleUserFormChoiceSelection()
var form = document.getElementById('form');
var optionsArray = CINodes[0]['choices'];
var optionNum;
var optionsHTML = "";
for (optionNum = 0; optionNum < optionsArray.length; optionNum++) {
	optionsHTML  += '<div class="input-container"><input id="' + optionNum + '" class="radio-button" type="radio" name="radio" value ="' + optionsArray[optionNum]['text'] + '"/><div class="radio-tile"><label for="' + optionNum + '" class="radio-tile-label">' + optionsArray[optionNum]['text'] + '</label></div></div>'
}
		form.innerHTML = optionsHTML;
updateCurrentState('switchQuestion', false);

// catches if user selects form option
$('#form input').on('change', function() {
	complexInnerStateHandleUserFormChoiceSelection(this.getAttribute('id'));
});

********************** Inner Loop Type 1 ***************************************
This inner loop displays linear questions one by one, not concurrently

//This should be placed within Phaser's "update" function
if (currentStateType == 'inner') {
	console.log("In Inner Loop");
	handleInner();
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Waiting a bit before handling to let the pretty animation happen...
async function innerStateHandleUserFormChoiceSelection() {
  await sleep(300);
  updateCurrentState('switchQuestion', true);
  if (currentState['questionsCount'] == currentState['numQuestions']) {
  	updateCurrentState('askQuestions', false);
  	deleteArticleAndForm();
  }
}

function handleInner() {
	if (currentState['loadedAssets'] == false) {

		var nextButton = game.add.button(0, 50, 'nextButton', nextState, this);
		nextButton.scale.setTo(.2, .2);

		// Note that if we don't want to load article (or ask questions, etc) immediately or at all we can definitely do that
		// Just add a variable in database, and act different based on it

		// We want to load an article
		updateCurrentState('loadArticle', true);
		// But we haven't yet
		updateCurrentState('loadedArticle', false);
		// We want to ask questions
		updateCurrentState('askQuestions', true);
		// And we want a new one right now
		updateCurrentState('switchQuestion', true);

		var currentStateId = stateArray[0];
		var articleId = states[currentStateId]['article'];

		// TODO: change updateCurrentState to take array of things to change,not
		// one at a time (also firebase let's us give up multiple things to 
		// update, so should be taking advantage of that...)
		updateCurrentState('prompts', articles[articleId]['questions']['prompt']);
		updateCurrentState('choices', articles[articleId]['questions']['choices']);
		updateCurrentState('numQuestions', currentState['prompts'].length);
		updateCurrentState('questionsCount', 0);				

		updateCurrentState('loadedAssets', true);
	}
	else {
		//relying on short-circuit evaluation: if we want to load an article and if it hasn't already been loaded, then load
		if (currentState['loadArticle'] && currentState['loadedArticle'] == false) {
			var currentStateId = stateArray[0];
			var articleId = states[currentStateId]['article'];
			var templateUsed = articles[articleId]['template']['num'];
			if (templateUsed == 0) loadTemplate0(articleId);
			updateCurrentState('loadedArticle', true);
		}

		// Once user choice detected, switchQuestion because true again. 
		// Once submit button is clicked, askQuestions becomes false 
		if (currentState['askQuestions'] && currentState['switchQuestion']) {
			innerStateLoadForm();
		}
	}
}

function innerStateLoadForm() {
	var form = document.getElementById('form');
	var optionsArray = currentState['choices'][currentState['questionsCount']];
	var optionNum;
	var optionsHTML = "";
	for (optionNum = 0; optionNum < optionsArray.length; optionNum++) {
		optionsHTML  += '<div class="input-container"><input id="' + optionNum + '" class="radio-button" type="radio" name="radio" value ="' + optionsArray[optionNum] + '"/><div class="radio-tile"><label for="' + optionNum + '" class="radio-tile-label">' + optionsArray[optionNum] + '</label></div></div>'
	}
 	form.innerHTML = optionsHTML;
	updateCurrentState('questionsCount', currentState['questionsCount'] + 1);
	updateCurrentState('switchQuestion', false);

	// catches if user selects form option
	$('#form input').on('change', function() {
		console.log("gotit");
		innerStateHandleUserFormChoiceSelection();
	});
}

********************************* Handling score changes ***********************
Assuming the basic inner loop where
function determineScore(userChosen) {
	var correct = articles[0]['answer'];
	if (correct == 'fake' && userChosen == 'fake') {
		alert("CORRECT!");
		return score + 1;
	}
	if (correct == 'real' && userChosen == 'real') {
		alert("CORRECT!");
		return score + 1;
	}
	alert("WRONG!");
	return score - 1;
}

function updateScore() {
	var newData = {
		score: score 
	};

	var updates = {};
	updates['/users/' + uid] = newData;

	firebase.database().ref().update(updates);
	scoreText.text = 'Score: ' + score;
}

function nextArticle() {
	var article = document.getElementById('article');
	article.style.display = 'none';
	articles.splice(0,1); 
	switchArticle = true;
}

function yesClicked(button) {
	score = determineScore('real');

	updateScore();

	nextArticle();
}

function noClicked(button) {
	score = determineScore('fake');

	 		updateScore();

	nextArticle();
}

