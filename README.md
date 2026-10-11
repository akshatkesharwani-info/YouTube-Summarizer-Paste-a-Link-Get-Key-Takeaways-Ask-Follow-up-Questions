# YouTube Summarizer — Paste a Link, Get Key Takeaways, Ask Follow-up Questions

A Colab notebook that downloads the transcript of a YouTube video, summarizes it (even very long ones), and lets you ask follow-up questions that are answered from the full transcript.

## Problem

Long videos take hours to watch just to find the main points, and a normal summary cannot answer the specific questions that come up afterwards.

## Approach

1. Get the video id from the link (normal, `youtu.be`, Shorts and embed links all work)
2. Download the transcript with `youtube-transcript-api`. If YouTube blocks it, the notebook asks you to paste the transcript, or uses a sample, so it never gets stuck
3. Split the transcript into parts of about 6,000 characters
4. Summarize each part with an LLM on Groq (`openai/gpt-oss-120b`). A failed call is retried up to 3 times and reported, so an error message never ends up in the summary
5. Combine the part summaries into a final output: summary, 5 key takeaways, who should watch
6. For follow-up questions, build a TF-IDF search index over the full transcript (2,500-character chunks, simple word stemming). The 4 best matching chunks plus the summary go to the model, which answers only from them

## Tech Stack

Python, youtube-transcript-api, scikit-learn (TF-IDF), Groq API (OpenAI-compatible client), Google Colab

## Results

One real run on a long conversation-style video:

| Step | Result |
|---|---|
| Transcript length | 164,332 characters |
| Parts summarized | 28 |
| Search chunks for follow-ups | 66 |
| Output | Summary, 5 takeaways, "who should watch" |
| Follow-up questions asked | 5, all answered, 4 transcript parts matched each time |

**Why follow-ups search the full transcript:** my first version answered follow-ups from the part summaries (cut at 12,000 characters). On the same video and the same question, "What does she say about how memory works in the brain?", the first version gave a general line about phones and cognitive training. The second version, searching the transcript, gave specifics about where memory is stored and why attention matters.

What I did **not** do: I did not score the answers against the video, so there is no accuracy number. The retry step did not trigger in my run, so it was only tested with simulated errors. The summary reports what the video says, not whether it is true.

## How to Run

1. Open `YouTube_Summarizer.ipynb` in Google Colab
2. Get a free API key from [console.groq.com](https://console.groq.com)
3. Click Runtime → Run all, paste the key when asked
4. Paste a YouTube link (or press Enter to use the sample transcript)
5. Ask follow-up questions in the last cell. Type `exit` to stop

## Limitations

- Needs captions. Videos without a transcript cannot be summarized unless you paste the text
- Tested on one long video, so results on other videos are not measured
- Follow-up search is keyword based. A question worded very differently from the video can miss the right part
- "This is not mentioned in the video" means the answer was not in the 4 best matching parts. It does not prove the video never says it
- Summaries depend on the transcript. Auto-generated captions can contain mistakes
- Runs as a notebook only. It is not deployed

## Next Steps

- Add timestamps to each takeaway
- Use embeddings for follow-up search so paraphrased questions match better
- Streamlit app. Note: YouTube often blocks transcript downloads from cloud servers, so a hosted version needs a paste-transcript option or a proxy

## Author

**Akshat Kesharwani** | [GitHub](https://github.com/akshatkesharwani-info) | [Portfolio](https://akshatkesharwani-info.github.io/)
