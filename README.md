# RAGProject CLI README

## What this project does
This .NET 8 console application ingests plain text stories, chunks them, generates embeddings, stores chunk embeddings on disk, then answers questions using only retrieved chunks as context.

## Requirements
1. .NET 8 SDK
2. A Gemini API key configured in GlobalSettings.cs
3. One or more .txt files under the Stories folder

## Project layout
Stories
Place source .txt files here.

VectorStore
Created automatically. One subfolder per ingested story file. Each subfolder contains json chunk files.

## Run
This project requires an API key for the LLM provider. Open `GlobalSettings.cs`, find the `API_KEY` constant, and replace it with your own key (default model is gemini-2.5-flash):

```text
public const string API_KEY = "YOUR_KEY_HERE";
```

Then copy txt files that LLM should ground on to 'Stories', which has three default files. 

Finally, from the project directory "RAGProject.Cli" that contains the csproj and the Stories folder:

```text
dotnet build
dotnet run
```

If you run from Visual Studio, set the working directory to the project folder. Paths for Stories and VectorStore are relative, so a different working directory can make the app look like it needs ingestion again.

## Phase 1 ingestion
1. The app lists all .txt files in Stories and prompts you to select which files to ingest by number.
2. For each selected file, the app creates a collection folder under VectorStore using the story file name.
3. The app chunks the document using SemanticSlicer with MaxChunkTokenCount 200 and OverlapPercentage 10.
4. Each chunk is embedded and saved as a json file under that collection folder.
5. The app waits briefly between chunks to reduce the chance of rate limiting.

Skip and partial ingestion behavior is per collection folder.
If the collection folder already contains more than 15 json files, ingestion for that story is skipped.
If the collection folder contains 1 to 15 json files, it is treated as partial ingestion, the folder is cleared, then the story is ingested again.

## Phase 2 question answering
1. The app lists all collection folders under VectorStore and prompts you to select which collections to answer from.
2. For each selected collection, it retrieves up to 5 relevant chunks, merges results, keeps the overall top 5, then applies a similarity threshold of 0.50.
3. The LLM is prompted to answer using only the retrieved context. If the answer is not in the retrieved context, it should refuse with the configured refusal sentence.

## Quick troubleshooting
If no stories are found, confirm the Stories folder exists in your working directory and contains .txt files.
If ingestion differs between VS Code and Visual Studio, confirm both are using the same working directory so they point to the same VectorStore folder.
