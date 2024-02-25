from flask import Flask
from flask import jsonify
from flask import request
from collections import Counter

app = Flask(__name__)

seen_strings = {}

@app.route('/')
def root():
    return '''
    <pre>
    Welcome to the Stringinator 3000 for all of your string manipulation needs.

    GET / - You're already here!
    POST /stringinate - Get all of the info you've ever wanted about a string. Takes JSON of the following form: {"input":"your-string-goes-here"}
    GET /stats - Get statistics about all strings the server has seen, including the longest and most popular strings.
    </pre>
    '''.strip()

@app.route('/stringinate', methods=['GET','POST'])
def stringinate():
    input = ''
    if request.method == 'POST':
        input = request.json['input']
    else:
        input = request.args.get('input', '')

    if input in seen_strings:
        seen_strings[input] += 1
    else:
        seen_strings[input] = 1

    processed_string = input.translate(str.maketrans('', '', ' \.,?!:;[]()')).lower()
    char_counts = {}
    for char in processed_string:
        char_counts[char] = char_counts.get(char, 0) + 1
    most_frequent_char = max(char_counts.items(), key=lambda item: item[1])


    return {
        "input": input,
        "length": len(input),
        "most_frequent_char": most_frequent_char
    }

@app.route('/stats')
def string_stats():
    # Ensure seen_strings is defined and holds input strings
    if not seen_strings:
        return jsonify({"inputs": [], "most_popular": None, "longest_input_received": None})

    # Calculate string lengths and counts
    string_lengths = {string: len(string) for string in seen_strings}
    string_counts = Counter(seen_strings)

    # Find longest string and most popular
    longest_string = max(string_lengths, key=string_lengths.get)
    most_popular_string, count = string_counts.most_common(1)[0]

    return jsonify({
        "inputs": seen_strings,
        "most_popular": most_popular_string,
        "longest_input_received": longest_string
    })