# ed-laravel-uuid-test

Recently I was working on a project where we issue API tokens to "machine users". A machine user represents a remote system that would interact with our API. While it lived in the users table like normal application users, it was set up in such a way that it could only authenticate using the API token. The machine user account had no password and no way to even attempt to reset the password.

Part of this setup involved generating a UUID as part of the machine user's "email address". Again, this wasn't meant to be a real email address. It just uniquely identified this machine user in audit logs.

While writing a test for the command which generates these machine users and issues them an API token, I needed to verify that the email address was generated as expected. Let's say the email looks like this:

machine-user-<uuid>@domain.disabled

How would you write a test to verify that when UUIDs are generated randomly? In the past, I've done this by simply testing the pattern with a regular expression, skipping over the characters from the random UUID.

But this time, I vaguely remembered seeing a recent Twitter exchange where someone mentioned test helpers specificaly for random methods on the Str class.

Sure enough, the Str::uuid docs mention the createUuidsUsing method, which was built for exactly this purpose. Here's how I used it in my test:

![image](https://github.com/GrytsenkoAndrey/ed-laravel-uuid-test/assets/63291871/0355e71a-fc18-4a98-b1fb-ba811810735b)

By using this helper method, I was able to control the UUID generation in my test. This made it easy to test the exact value, making a cleaner and more readable test than my old regex-based approach.

There is one caution to keep in mind when choosing a fake UUID value to assert against: Don't always use the same value. If every single one of my tests always used the same fake value, it would be possible to have false positives in some edge cases. While it may not be likely, especially if you're using the RefreshDatabase trait, I prefer to play it safe and mix up the fake values from test to test.

Finally, don't forget to reset this test helper with Str::createUuidsNormally. Without it, all the UUIDs generated for the rest of my test suite would continue to match my faked value. In fact, this is why I like to call this clean-up method even before I make my test assertions. In case an assertion fails, I'd prefer to not have the rest of my UUIDs faked if the test suite continues to run.
