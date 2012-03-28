---
layout: post
title: "Simple Dependency Injection and MiniTest::Mock"
date: 2012-03-28 19:53
comments: false
---

I recently wrote a [Ruby client for Amazon Alexa's APIs][1], and thought I'd
pull out an example of nice, simple dependency injection to facilitate unit
testing. Nothing revolutionary or complicated, just good practice.


The example is based around a `UriSigner` class, normally used by the calling
code like this:

    UriSigner.new(*credentials).sign_uri(uri)

The calling code doesn't know or care that `UriSigner` depends on [Base64][2],
[OpenSSL::Digest::SHA256][3] and [OpenSSL::HMAC][4]. The unit test for
`UriSigner`, however, cares for two reasons.

Firstly, they're external dependencies, and only need to be tested for correct
usage, not correct implementation.

Secondly, these dependencies represent an encoder and a cryptographic hash
function; they're deterministic, but they return very opaque data which can
make tests and their failure messages difficult to understand.

So instead of testing against magical (computed in advance) Base64 strings and
HMAC hashes, I've used simple `attr_writer` dependency injectors:

{% gist 2224862 uri_signer.rb %}

The unit test can then inject [MiniTest::Mock][5] instances in place of the
real Base64 and HMAC implementations, setting expected method calls and their
return values:

{% gist 2224862 uri_signer_spec.rb %}

As simple as that.

The same approach is used by the HTTP Client class to stub out actual HTTP
calls via Net::HTTP. There's great libraries like [VCR][6], [WebMock][7] and
[FakeWeb][8] for handling this, but sometimes it's easier to keep it lo-fi:

{% gist 2224862 client.rb %}
{% gist 2224862 client_spec.rb %}

This kind of dependency injection is one of many basic techniques which aren't
fancy enough to get a lot of press, but go a long way to keeping your objects
and tests in order.

Got any thoughts? Hit me up, I'm [@pda on Twitter](https://twitter.com/pda),
where I generally write about this kind of thing.


[1]: https://github.com/flippa/ralexa
[2]: http://ruby-doc.org/stdlib-1.9.3/libdoc/base64/rdoc/Base64.html
[3]: http://ruby-doc.org/stdlib-1.9.3/libdoc/openssl/rdoc/OpenSSL/Digest.html
[4]: http://ruby-doc.org/stdlib-1.9.3/libdoc/openssl/rdoc/OpenSSL/HMAC.html
[5]: http://ruby-doc.org/stdlib-1.9.3/libdoc/minitest/mock/rdoc/MiniTest/Mock.html
[6]: https://github.com/myronmarston/vcr
[7]: https://github.com/bblimke/webmock
[8]: https://github.com/chrisk/fakeweb
