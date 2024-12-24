### Project Description
This project is a Movie Nerd Recommender chatbot agent. The bot knows everything there is to know about movies, and it is respectful and enthusiastic. If the user is not asking specific questions, the movie nerd will offer assistance.

Generally the ChatBot will output a list of movie suggestions that coincide with what the user queried.

### Implementation Details

The data used to build ghe Vector Database was a subset of 5000 movies on an Imdb dataset from (Kaggle)[https://www.kaggle.com/datasets/amanbarthwal/imdb-movies-data]. The input json objects include the following information:
* Title: name of the movie.
* Year: year the movie was released.
* Rating: age rating given to the movie. 
* Duration: length of the movie in minutes.
* Genre: genre(s) of the movie
* Imdb Rating: IMDB user rating for the movie.
* Metascore: score from critics.
* Director: director(s) of the movie.
* Cast: main actors in the movie.
* Description: brief summary of the movie's plot.


Moderation:
- Used the OpenAI moderations API to ensure that user prompts were not harmful

Prompt injection prevention:
- Employed the use of delimiters and pre-prompt instruction to ensure that any mal-intented input to the model will be disregarded

ChatBot context:
- Used a system prompt that detailed general instructions for how the bot should behave.
- Used chain-of-thought prompting and specified that it should output HTML format so the output on the UI could be displayed in a user friendly way.
- After gathering the relevant vectors from the database, this information is appended as a system prompt to the context.
- Then the user's prompt and assistant's response is appended to the context, so that the bot has a sense of 'memory'.

#### Vector database and LLM cross-referencing implementation details

Handling weaviate client:
- Created a function to handle connecting to already existing weaviate client or creating the client if it did not exist locally
- Used flasks teardown_appcontext functionality to ensure the weaviate client was closed after the application is stopped.

Creating the vector database:
- Used a function to drop and recreate the collection in the case that changes are made to the vector database and it needs to be re-created.
- Conditionally either accessed the already existing database if it existed or created the collection if not
- Used descriptive property names to store things about the movie like the Imdb rating so that the vector database could accurately find near text when prompted by the user. 

Querying the database:
- Used the near_text search operator to search the database with the users input as natural language, and set specific settings to ensure only relevant data was returned from the vector database (ie setting a minimum certainty of 0.7)

Cross-referencing the vector database results with the LLM:
- For each user input, the 5 most similar vectors in the database are retrieved and passed to the LLM to check if the vector db results retrieved are consistent with what the user was querying.
- To do this, the LLM is instructed to respond with 'yes' or 'no' indicating if the vector db result is a 'good' answer for what the user wanted.
- Only the vector db results that goet a "yes" are passed to the main LLM query, where they are provided as a system role to the chatbot context, and the LLM is told that these results may be useful, but are not binding.