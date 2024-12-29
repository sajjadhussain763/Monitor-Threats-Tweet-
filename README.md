# Monitor-Threats-Tweet-
import tweepy
import csv
from textblob import TextBlob  # For sentiment analysis
import re  # For extracting URLs
import matplotlib.pyplot as plt  # For plotting graphs
from wordcloud import WordCloud  # For generating the tag cloud

# Twitter API credentials (from the user's provided keys)
API_KEY = 'avTBuutWl9lRSCIgfkpBvBYRK'
API_SECRET_KEY = 'ck0V2WIspuRZNE9CdWE0kePTrXRiujTVzJGTnbVopxoPMsxgD8'
ACCESS_TOKEN = '1871235200136691714-fszO1jDRflPhib3FzIGfEDCTX7243A'
ACCESS_TOKEN_SECRET = 'uQCLdmhyeSYKZo6pcpN3IQURWtHbn3dVdLfdhvEFs08PL'
BEARER_TOKEN = 'AAAAAAAAAAAAAAAAAAAAADfUxgEAAAAAdVqUEvkHEzy9VjGfHBH2hqOQm2U%3DbnNFlXcyh9SiDG8ojkevluAnYklm3xCIBV8cAtSVLC6sCw2u9k'

# Authenticate using Bearer Token for Twitter API v2
def authenticate_twitter():
    try:
        client = tweepy.Client(
            bearer_token=BEARER_TOKEN,
            consumer_key=API_KEY,
            consumer_secret=API_SECRET_KEY,
            access_token=ACCESS_TOKEN,
            access_token_secret=ACCESS_TOKEN_SECRET
        )
        print("Authentication successful")
        return client
    except tweepy.errors.TweepyException as e:
        print(f"Error during authentication: {e}")
        return None

# Analyze sentiment of a tweet using TextBlob
def analyze_sentiment(tweet_text):
    analysis = TextBlob(tweet_text)
    if analysis.sentiment.polarity > 0:
        return 'Positive'
    elif analysis.sentiment.polarity == 0:
        return 'Neutral'
    else:
        return 'Negative'

# Extract URLs from tweet text
def extract_urls(text):
    return re.findall(r'https?://\S+', text)

# Fetch and analyze tweets
def fetch_and_analyze(client, keywords):
    try:
        # Construct query for Twitter API v2
        query = ' OR '.join(keywords)
        response = client.search_recent_tweets(query=query, tweet_fields=['author_id', 'text'], max_results=10)

        # Check if tweets were fetched successfully
        if not response.data:
            print("No tweets found for the given query.")
            return False, []

        # Open CSV file to save results
        results = []
        tweet_texts = []  # List to store tweet texts for the tag cloud
        with open('tweets_analysis.csv', mode='w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file)
            writer.writerow(["Author ID", "Tweet Text", "Sentiment", "URLs"])

            # Analyze tweets and save results
            for tweet in response.data:
                sentiment = analyze_sentiment(tweet.text)
                urls = extract_urls(tweet.text)
                results.append({'sentiment': sentiment, 'urls': urls})
                tweet_texts.append(tweet.text)  # Add tweet text for tag cloud
                writer.writerow([tweet.author_id, tweet.text, sentiment, ", ".join(urls)])
                print(f"Author ID: {tweet.author_id} | Sentiment: {sentiment} | URLs: {', '.join(urls)}")

        print("Tweets fetched, analyzed, and saved to 'tweets_analysis.csv' successfully.")
        return results, tweet_texts

    except tweepy.errors.Forbidden as e:
        print(f"Access denied: {e}")
        return False, []
    except tweepy.errors.TweepyException as e:
        print(f"Error while fetching tweets: {e}")
        return False, []

# Generate and plot the tag cloud
def generate_tag_cloud(tweet_texts):
    # Combine all tweet texts into one large string
    all_text = " ".join(tweet_texts)
    
    # Generate word cloud
    wordcloud = WordCloud(width=800, height=400, background_color='white').generate(all_text)
    
    # Display the word cloud
    plt.figure(figsize=(10, 6))
    plt.imshow(wordcloud, interpolation='bilinear')
    plt.axis('off')
    plt.title('Tag Cloud of Tweet Keywords')
    plt.show()

# Plotting the sentiment and URL results in graphs
def plot_results(results):
    sentiments = [entry['sentiment'] for entry in results]
    url_counts = [len(entry['urls']) > 0 for entry in results]  # True if URL exists

    # Plot sentiment distribution (bar chart)
    plt.figure(figsize=(14, 6))

    # Sentiment analysis distribution
    plt.subplot(1, 2, 1)
    sentiment_counts = {'Positive': sentiments.count('Positive'), 'Neutral': sentiments.count('Neutral'), 'Negative': sentiments.count('Negative')}
    plt.bar(sentiment_counts.keys(), sentiment_counts.values(), color=['green', 'blue', 'red'])
    plt.title("Sentiment Distribution")
    plt.xlabel("Sentiment")
    plt.ylabel("Count")

    # URL presence distribution (pie chart)
    plt.subplot(1, 2, 2)
    url_presence = {'With URLs': sum(url_counts), 'Without URLs': len(url_counts) - sum(url_counts)}
    plt.pie(url_presence.values(), labels=url_presence.keys(), autopct='%1.1f%%', startangle=90, colors=['yellow', 'orange'])
    plt.title("URL Presence in Tweets")

    plt.tight_layout()
    plt.show()

# Main function
def main():
    print("Fetching and analyzing tweets...")

    # Authenticate
    client = authenticate_twitter()
    if client:
        # Define keywords for search
        keywords = ['cyber attacks', 'phishing attacks', 'malware attacks', 'Doos attacks']

        # Fetch and analyze tweets
        results, tweet_texts = fetch_and_analyze(client, keywords)

        if results:
            print("Tweets fetched and analyzed successfully.")
            plot_results(results)  # Plot graphs
            generate_tag_cloud(tweet_texts)  # Generate and display tag cloud
        else:
            print("Failed to fetch tweets.")
    else:
        print("Authentication failed.")

if __name__ == "__main__":
    main()
