# The No Database URL Shortener

This is a simple little URL shortener for personal use which utilizes flat files rather than a database in which to store its data. It's easy to set up, easy to use and secure.

## Installation

To install the No DB URL Shortener, take the following steps:

* Upload this entire repository to the host on which you want this to live. It should work on the document root or in a subdirectory. Don't forget the `.htaccess` file. Alternatively, you can do clone this repository to the server where you want this to live.
* In the `inc` directory, rename `config.php.sample` to `config.php` and change the values in that file to something that works for you.
* Change permissions on `inc/config.php`, `inc/daily-count.txt` and `content/urls` to be writable by the server. Ususally, that involves a `chmod 777` on those files and directories in the terminal.
* Hit the URL you've uploaded this to in your browser. You should be prompted to set a password, so do so!

That's it! Once you've set your password, the URL Shortener is ready for you to use.

## Adding a Short URL

To generate a short URL and add it to the system, there is **/create** URL which you can make a REST call to. It accepts the following parameters:

**pw** _required_  
This is the password you set up during the installation process. Any requests to the create endpoint that don't include the password will result in an error.

**link** _required_  
This is the link that you'd like to shorten.

**id** _optional_  
If you'd like the hash to be generated from an ID (say, the ID of a post in your CMS), you can pass that id in here. If omitted, an id will be calculated utilizing the current date and the number of times you created a shortlink that day, and that will be used to generate the hash.

**hash** _optional_  
If you have a hash, or slug, that you want to utilize, you can pass it in here. This is handy if you want create vanity shortlinks, such as `http://mub.io/awesome`. In that case, you'd add `hash=awesome` to the create request. Note that if a hash is already in use in the system, what you pass in here will be ignored and it will generate a hash for you.

When you make a call to **/create**, you will get a JSON response. A successful response will look something like this:

```
{
    "originalURL": "http://michaelbester.com",
    "shortURL": "http://mub.io/olL43",
    "baseURL": "mub.io",
    "hash": "olL43"
}
```

If an error is encountered, the JSON will return an appropriate error message, something like so:

```
{
    "error": "Sorry, but your password was incorrect."
}
```

### Example REST call function

Here's an example PHP function that makes the REST call to the URL shortener, passing in parameters from the `$_POST` array. It has the `pw` parameter baked in, but you'd still have to pass in at least `link` as a post parameter:

```
public function build_short_url()
{
  $url_shortener_endpoint = 'http://mub.io/create;
	$url_shortener_password = 'abc123';

	$post = array_merge($_POST, array('pw' => $url_shortener_password));

	$options = array(
		CURLOPT_POST => 1,
		CURLOPT_HEADER => 0,
		CURLOPT_URL => $url_shortener_endpoint,
		CURLOPT_FRESH_CONNECT => 1,
		CURLOPT_RETURNTRANSFER => 1,
		CURLOPT_FORBID_REUSE => 1,
		CURLOPT_TIMEOUT => 4,
		CURLOPT_POSTFIELDS => http_build_query($post)
	);

	$ch = curl_init();
	curl_setopt_array($ch, $options);
	if ( ! $result = curl_exec($ch ) ) {
		$content = new stdClass();
		$content->error = "curl_error($ch)";
		$content = json_encode($content);
	}
	curl_close($ch);
	echo $result;
}
```

Note that I currently haven't provided any alternative methods to create new short URLs such as a form or a bookmarklet. The current implementation suits my immediate needs, but if you would like to set up some alternative avenues to create URLs, pull requests are more than welcome.

## Version History

0.1.0 - Current Stable

## Contact Me

You can contact me at my site, [http://michaelbester.com](http://michaelbester.com), as well as [on Twitter](http://twitter.com/mibester).

## License

MIT License. See the `LICENSE` file. You can use `URLShortener` in personal projects, open source projects and commercial products.

