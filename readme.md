# Python OAuth 2.0

## Introduction

This is an implementation of the OAuth 2.0 spec in Python for servers and clients. Originally when I set out to implement OAuth 2.0 in Python I pulled up [the spec](http://tools.ietf.org/html/draft-ietf-oauth-v2-22) and started writing specs for which I would later write tests.

Unfortunately, I fell asleep with boredom. So what I'm going to instead do is write the an implementation that covers the minimum viable product, and as the spec reaches completion, so shall I, and hopefully the community, fill out some of the darker corners in this library.

What is the minimum viable product, you might be wondering?

## Minimum Viable Product

As of this writing, this library will provide:

1. Extensible primitives to support core OAuth protocol
2. A way to implement the most common endpoints using those primitives
3. A path forward to flesh out the rest of the spec

##### So what is that really?

Basically, you will be able to implement an OAuth server similar to what [Facebook](http://developers.facebook.com/docs/authentication/), [Dailymotion](http://www.dailymotion.com/doc/api/authentication.html), [Instagram](http://instagram.com/developer/auth/), [and](http://gowalla.com/api/docs/oauth) [others](http://code.google.com/apis/accounts/docs/OAuth2.html) offer with their services since they too cover the minimum viable product.

## So Let's Get Started

Here is how easy it is to get an OAuth provider up and running:

	from oauh20 import Server, Client, Token
	
	# create a default Client
	client = oauth20.Client.storage.create(
		client_id='1234',
		client_secret='abcd1234',
		client_type='server',
		redirect_uris=['http://example.com/redirect/'])
	 
	# create a default server
	server = oauth20.Server()
	
	# an example http server implementation
	from wsgiref.simple_server import make_server
	from wsgiref.util import request_uri

	def simple_server(environ, start_response):
		route = lambda u: request_uri(environ).endswith(u)
		headers = [('Content-type', 'text/plain')]
		val = ''
		status = '200 OK
		if route('/oauth/authorize'):
			code, error = server.authorize(environ)
			status = '302 Found'
			headers += [('Location', server.redirect_uri(code, error))]
		elif route('/oauth/token'):
			token, error = server.get_token(environ)
			val = server.ger_urlencoded_token(token, error)
		elif route('/protected_resource'):
			if not server.validate(environ):
				val = 'sorry :('
				status = '401 Unauthorized'
			else:
				val = 'accessing protected resource'
		elif route('/unprotected_resource'):
			val = 'oh, hello there.'
		
		start_response(status, headers)
		return val
	
	httpd = make_server('', 8000, simple_server)
	httpd.serve_forever()

Keep in mind that such a provider provides limited functionality, however implements most of what we are trying to provide in an entirely tolerable amount of code. When using only the `Server`, `Code`, and `Token` primitives, all information is stored in memory and is entirely vacuated upon each server restart.

