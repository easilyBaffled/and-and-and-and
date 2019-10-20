![Travis (.com)](https://img.shields.io/travis/easilyBaffled/and-and-and-and?style=for-the-badge)
# & and && and ;
> Bash and Regex are everywhere, and they are not going away, might as well learn them.

I was setting up the continuous integration for a new project and once again found myself copy-and-pasting code that I didn’t grasp. This time it came from [Cypress.io](https://docs.cypress.io/guides/guides/continuous-integration.html#Solutions) to run an app’s integration tests on a remote server.

```bash
> npm start & wait-on http://localhost:8080
> cypress run
```     

I knew the `&` was critical, but that was the extent of my understanding. I have a feeling that I am not alone. Let’s’ fix that. Shall we?

For educational purposes, we’ll be using
```bash
> http-server & wait-on http://localhost:8080 && cypress run; kill $!
```
If you’d like to follow along I have working examples in [easilyBaffled/and-and-and-and](https://github.com/easilyBaffled/and-and-and-and) and running on [Travis CI](https://travis-ci.org/easilyBaffled/and-and-and-and).

TL;DR: Skip to **Bash Connections**
Real TL;DR: go to [explainshell.com](https://explainshell.com/explain?cmd=http-server+%26+wait-on+http%3A%2F%2Flocalhost%3A8080+%26%26+cypress+run%3B+kill+%24%21)

Before diving into the Bash syntax, let’s get the Node modules out of the way. For simplicity’s sake, assume everything works.

## Node Processes 
`http-server` is an NPM module that serves a tiny web app out of the box. Once the server starts, it’ll continually run on the thread, blocking any other work. Not ideal when we also want to run our tests.

`wait-on http://localhost:8080` stalls the thread while `http-server` prepares and serves the page. Our server isn’t going to give us a programmatic indication that the page is ready, so if we didn’t use these tests would run too early and fail.

`cypress run` runs our tests in headless mode. 

## Bash Connections
`&`  executes the command on the left-hand side in the background. By running `http-server &`, the server will no longer be blocking the thread, and we can move onto `wait-on`.

`&&` works similar to the same operator in JavaScript. The operation on the right-hand side only runs if the operation on the left-hand side returns `true` or in the case of  Bash exit status of `0`.  `wait-on` returns that `0` when the page is ready, which clears the way for `cypress run`. 

`;`  is the standard command separator. Unlike `&&` , `;` doesn’t require a specific exit status. Our example would still work if we used `;` instead of `&&`, but that _is_ assuming everything goes as planned

`kill`  kills a process with a given process Id.

`$1` is a special character that holds the process Id of the most recently backgrounded process. For us, that is our server. So when tied with `kill`, it kills the backgrounded server _after_ the tests have finished.

## Wrap up 
Now that you understand a little more about Bash, there are tools like  [start-server-and-test](https://github.com/bahmutov/start-server-and-test)  and [concurrently  -  npm](https://www.npmjs.com/package/concurrently) that can simplify this sort of work, so you don’t need `&`, `&&`, `;`. I’m still getting a solid grip on these things, so if you have caught something that I missed or have additional examples/usage please leave a comment. I’d love to get better at this. 
