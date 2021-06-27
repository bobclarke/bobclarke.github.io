---
layout: post
title: JWT Token generation and verification using PKI
date: 2019-09-29 20:18:21.000000000 +01:00
type: post
---
<p><!-- wp:paragraph --></p>
<p>In this exercise we'll use two separate browser windows on <a href="https://jwt.io/">https://jwt.io</a>. In window1 we’ll be generating our JWT token and in window2 we’ll be verifying it.  Instead of a shared secret we'll be using asymmetric keys.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>A great learning source for JWT can be found here <a href="https://www.youtube.com/watch?v=7Q17ubqLfaM&amp;t=588s">https://www.youtube.com/watch?v=7Q17ubqLfaM&amp;t=588s</a></p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Generate a private&nbsp;key&nbsp;as&nbsp;follows:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:preformatted --></p>
<pre class="wp-block-preformatted">openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048</pre>
<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p>Now extract the public key </p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:preformatted --></p>
<pre class="wp-block-preformatted">openssl rsa -pubout -in private_key.pem -out public_key.pem</pre>
<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Now carry out the following steps in window1 </p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>set the header as {"alg": "RS256”, "typ": "JWT”}</li>
<li>set the payload as {"sub": "1234567890”, "name": "Bob Clarke”, "iat": 1516239022}</li>
<li>Paste the private and public keys into the relevant windows (including the BEGIN and END  markers) - NOTE, at this stage you don’t actually require the public key as we’re only generating the token and not validating it. </li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:paragraph --></p>
<p>This will generate the following token:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:html --></p>
<div style="padding:3px;width:95%;word-wrap:break-word;font-size:16px;">
<font color="red">eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9</font>.<font color="purple">eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkJvYiBDbGFya2UiLCJpYXQiOjE1MTYyMzkwMjJ9</font>.<font color="#3185FC">An3zJCNTsAc0w1Lx8vEcBWiPOsJVOrtyFJFgHPKvn64iTvDvY9yDABjKS3MGLZtboLoT62K1LsqA0MyZA_xLTObAwMsc_agg1U49MDREtaGsmaakaRozVtysOM4P0ixCDrbhlYKfS2jL0uUgRwDVEw6AuSl3ZZHPCp1dS0FMtiEu_YFHTG4wy_qt7WO5KufvZ1ftFHLbXXjiQk7L4M2JL7iuzFRMJIlrWpsJisKtyJzdjocE0PKLqQJV8DjLYtGm_DYvSpnaGN9tf6A9KtFybEwg0d6bOMvW7fK7SE0W4YMtk32Sj3X3aXT0UW_ztbymDVd4-i8HMawfIh_2SKKFDw</font>
</div>
<p><!-- /wp:html --></p>

The diagram below shows how a JWT Token is&nbsp;generated:
<img class="polaroid" src="/assets/images/jwt1.png"/>

<br>

It’s&nbsp;important to note that this&nbsp;token is not secret, i.e it’s just base64 encoded and therefore anyone can take the red part&nbsp;and&nbsp;simply run it through base64 -D to decode it (example below):



{%highlight bash%}
echo "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkJvYiBDbGFya2UiLCJpYXQiOjE1MTYyMzkwMjJ9" | base64 -D ; echo 
#=>{"sub":"1234567890","name":"Bob Clarke","iat":1516239022}
{%endhighlight%}

So, secrecy is not what we’re after here, what we’re after is message integrity, i.e. making sure not has not been tampered with.   To verify message integrity, carry out the following steps in window2:
* paste the token into the "Encoded" box on the left
* You’ll notice that the header and payload are decoded normally (i.e, as mentioned above, secrecy is not the aim here) however you’ll also see that there’s a big red message at the bottom saying Invalid Signature (see image below)  

<img class="polaroid" src="/assets/images/jwt2.png"/>

<br>

To verify whether the message is valid (i.e that it has not been tampered with) we need to paste in public key. The process for token verification is shown in the diagram below:

<img class="polaroid" src="/assets/images/jwt3.png"/>

<br>
And here’s the result in window2 with the signature verified and therefore message integrity:
<img class="polaroid" src="/assets/images/jwt4.png"/>

<br>

<a href="https://www.toptal.com/devops">Toptal ref</a>


