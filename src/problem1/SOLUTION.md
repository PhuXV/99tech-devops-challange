# Solution

## Command

<pre>
grep '"symbol": "TSLA"' ./transaction-log.txt | grep '"side": "sell"' | jq -r '.order_id' | xargs -I{} curl -s "https://example.com/api/{}" >> ./output.txt
</pre>


## Explanation
This command pipeline performs the following steps:

Filter TSLA transactions:
<pre>
grep '"symbol": "TSLA"' ./transaction-log.txt
</pre>
Searches for all lines containing "symbol": "TSLA" in the transaction log file

Filter sell orders:
<pre>
grep -w '"side": "sell"'
</pre>

Filters the TSLA transactions to only include sell orders

Extract order IDs:

<pre>
jq -r '.order_id'
</pre>
Uses jq to parse each JSON line and extract just the order_id value
The -r flag outputs raw strings (without quotes)

Execute HTTP requests:
<pre>
xargs -I{} curl -s "https://example.com/api/{}"
</pre>
Takes each order ID and substitutes it into the URL using {} as placeholder  
```curl -s``` makes silent requests to avoid progress output

The URL is quoted to handle any special characters in order IDs

Write output:

<pre>
>> ./output.txt
</pre>
Appends all responses to the output file