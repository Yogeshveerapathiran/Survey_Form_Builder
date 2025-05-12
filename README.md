# Survey Form Builder (ReactJS)
## Date: 12/05/2025

## AIM
To create a Survey Form Builder using ReactJS..

## ALGORITHM
### STEP 1 Initialize States
mode ← 'build' (default mode)

questions ← [] (holds all survey questions)

currentQuestion ← { text: '', type: 'text', options: '' } (holds input while building)

editingIndex ← null (tracks which question is being edited)

responses ← {} (user responses to survey)

submitted ← false (indicates whether survey was submitted)

### STEP 2 Switch Modes
Toggle mode between 'build' and 'fill' when the toggle button is clicked.

Reset submitted to false when entering Fill Mode.

### STEP 3 Build Mode Logic
#### a. Add/Update Question
If currentQuestion.text is not empty:

If type is "radio" or "checkbox", split currentQuestion.options by commas → optionsArray.

Construct a questionObject:

If editingIndex is not null, replace question at that index.

Else, push questionObject into questions.

Reset currentQuestion and editingIndex.

#### b. Edit Question
Set currentQuestion to the selected question’s data.

Set editingIndex to the question’s index.

#### c. Delete Question
Remove the selected question from questions.

### STEP 4 Fill Mode Logic
#### a. Render Form
For each question in questions:

If type is "text", render a text input.

If type is "radio", render radio buttons for each option.

If type is "checkbox", render checkboxes.

#### b. Capture Responses
On input change:

For "text" and "radio", update responses[index] = value.

For "checkbox":

If option exists in responses[index], remove it.

Else, add it to the array.

### STEP 5 Submit Form
When user clicks "Submit":

Set submitted = true.

Display a summary using the questions and responses.

### STEP 6 Display Summary
Iterate over questions.

Display the response from responses[i] for each question:

If it's an array, join it with commas.

If empty, show "No response".


## PROGRAM
```SurveyApp.jsx
import React, { useState } from 'react';

export default function SurveyApp() {
  const [mode, setMode] = useState('build');
  const [questions, setQuestions] = useState([]);
  const [newQuestion, setNewQuestion] = useState({ text: '', type: 'text', options: '' });
  const [responses, setResponses] = useState({});
  const [submitted, setSubmitted] = useState(false);

  const handleAddQuestion = () => {
    if (!newQuestion.text) return;
    const optionsArray = ['radio', 'checkbox'].includes(newQuestion.type)
      ? newQuestion.options.split(',').map(opt => opt.trim())
      : [];
    setQuestions(prev => [
      ...prev,
      {
        ...newQuestion,
        options: optionsArray,
        id: Date.now(),
      },
    ]);
    setNewQuestion({ text: '', type: 'text', options: '' });
  };

  const handleDelete = id => {
    setQuestions(prev => prev.filter(q => q.id !== id));
  };

  const handleChangeResponse = (id, value) => {
    setResponses(prev => ({ ...prev, [id]: value }));
  };

  const handleCheckboxChange = (id, option) => {
    setResponses(prev => {
      const current = prev[id] || [];
      return {
        ...prev,
        [id]: current.includes(option)
          ? current.filter(o => o !== option)
          : [...current, option],
      };
    });
  };

  const handleSubmit = () => {
    setSubmitted(true);
  };

  return (
    <div style={{ textAlign: 'center' }}>
      <button
        onClick={() => {
          setMode(mode === 'build' ? 'fill' : 'build');
          setSubmitted(false);
        }}
        style={{ padding: '8px 16px', backgroundColor: '#3b82f6', color: 'white', border: 'none', borderRadius: '4px', marginBottom: '16px' }}
      >
        Switch to {mode === 'build' ? 'Fill' : 'Build'} Mode
      </button>

      {mode === 'build' ? (
        <div>
          <h2>Build Survey</h2>
          <input
            placeholder="Question text"
            value={newQuestion.text}
            onChange={e => setNewQuestion({ ...newQuestion, text: e.target.value })}
            style={{ margin: '5px', padding: '5px' }}
          />
          <select
            value={newQuestion.type}
            onChange={e => setNewQuestion({ ...newQuestion, type: e.target.value })}
            style={{ margin: '5px', padding: '5px' }}
          >
            <option value="text">Text</option>
            <option value="radio">Radio</option>
            <option value="checkbox">Checkbox</option>
          </select>
          {['radio', 'checkbox'].includes(newQuestion.type) && (
            <input
              placeholder="Options (comma separated)"
              value={newQuestion.options}
              onChange={e => setNewQuestion({ ...newQuestion, options: e.target.value })}
              style={{ margin: '5px', padding: '5px' }}
            />
          )}
          <button onClick={handleAddQuestion} style={{ margin: '5px', padding: '5px 10px', backgroundColor: '#10b981', color: 'white', border: 'none' }}>
            Add Question
          </button>

          <ul style={{ marginTop: '10px' }}>
            {questions.map((q, idx) => (
              <li key={q.id} style={{ marginBottom: '10px' }}>
                <strong>Q{idx + 1}:</strong> {q.text} ({q.type})
                {q.options?.length > 0 && (
                  <div style={{ marginLeft: '20px', fontSize: 'small' }}>
                    Options: {q.options.join(', ')}
                  </div>
                )}
                <button
                  onClick={() => handleDelete(q.id)}
                  style={{ color: 'red', marginLeft: '10px' }}
                >
                  Delete
                </button>
              </li>
            ))}
          </ul>
        </div>
      ) : (
        <div>
          <h2>Fill Survey</h2>
          {questions.length === 0 ? (
            <p>No questions added yet.</p>
          ) : submitted ? (
            <div>
              <h3>Your Responses</h3>
              <ul>
                {questions.map(q => (
                  <li key={q.id} style={{ marginBottom: '10px' }}>
                    <strong>{q.text}</strong><br />
                    {Array.isArray(responses[q.id])
                      ? responses[q.id].join(', ')
                      : responses[q.id] || 'No response'}
                  </li>
                ))}
              </ul>
            </div>
          ) : (
            <form
              onSubmit={e => {
                e.preventDefault();
                handleSubmit();
              }}
            >
              {questions.map(q => (
                <div key={q.id} style={{ marginBottom: '15px' }}>
                  <label><strong>{q.text}</strong></label><br />
                  {q.type === 'text' && (
                    <input
                      type="text"
                      onChange={e => handleChangeResponse(q.id, e.target.value)}
                      style={{ width: '100%', padding: '5px' }}
                    />
                  )}
                  {q.type === 'radio' &&
                    q.options.map(opt => (
                      <label key={opt} style={{ marginRight: '10px' }}>
                        <input
                          type="radio"
                          name={q.id}
                          value={opt}
                          onChange={() => handleChangeResponse(q.id, opt)}
                        />{' '}
                        {opt}
                      </label>
                    ))}
                  {q.type === 'checkbox' &&
                    q.options.map(opt => (
                      <label key={opt} style={{ display: 'block' }}>
                        <input
                          type="checkbox"
                          checked={responses[q.id]?.includes(opt) || false}
                          onChange={() => handleCheckboxChange(q.id, opt)}
                        />{' '}
                        {opt}
                      </label>
                    ))}
                </div>
              ))}
              <button type="submit" style={{ padding: '8px 16px', backgroundColor: '#2563eb', color: 'white', border: 'none' }}>
                Submit
              </button>
            </form>
          )}
        </div>
      )}
      <footer style={{ marginTop: '2rem', fontSize: '0.875rem', color: '#6b7280' }}>
        Created by Yogesh V.S. | Reg No: 212222040185
      </footer>
    </div>
    
  );
}
```
```index.js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```
```App.js
import React from 'react';
import SurveyApp from './components/SurveyApp';

function App() {
  return (
    <div className="App">
      <SurveyApp />
    </div>
  );
}

export default App;
```
## OUTPUT
![WhatsApp Image 2025-05-12 at 13 22 36_49baef3f](https://github.com/user-attachments/assets/70709094-d803-4bd6-b402-538058fbde15)
![WhatsApp Image 2025-05-12 at 13 22 39_2dc57eaa](https://github.com/user-attachments/assets/42f15993-3232-4760-9dc9-40ba614a7527)
![WhatsApp Image 2025-05-12 at 13 29 47_a12f81ce](https://github.com/user-attachments/assets/515d827e-c554-47f5-bf1a-8a85fe8712b6)


## RESULT
The program for creating Survey Form Builder using ReactJS is executed successfully.
