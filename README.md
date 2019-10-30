# Just a little Bash, & and && and More!

I was setting up continuous integration for a project and once again found myself copy-and-pasting command line code that I didn't quite grasp. This time it came from [Cypress.io](https://docs.cypress.io/guides/guides/continuous-integration.html#Solutions) to run integration tests on a remote server:

```bash
> npm start & wait-on http://localhost:8080
> cypress run
```     

I knew the `&` was critical, but I didn’t understand why. I have a feeling that I am not alone. So let's' fix that.

For educational purposes, we'll be using:
```bash
> http-server & wait-on http://localhost:8080 && cypress run; kill $!
```

If you'd like to follow along, I have examples you can play with in [easilyBaffled/and-and-and-and](https://github.com/easilyBaffled/and-and-and-and) and running on [Travis CI](https://travis-ci.org/easilyBaffled/and-and-and-and).

TL;DR: go to [explainshell.com](https://explainshell.com/explain?cmd=http-server+%26+wait-on+http%3A%2F%2Flocalhost%3A8080+%26%26+cypress+run%3B+kill+%24%21)

Before diving into the harder Bash syntax, let's get the Node modules out of the way. For simplicity's sake, assume everything works.

## Node Processes 

`http-server` is an NPM module that serves a tiny web app out of the box. Once the server starts, it continually runs on the thread, blocking any other work. If you’re working locally, that isn’t an issue because you can open a new tab in the terminal, but the goal here is to run this and our tests on a remote CI server.

`wait-on http://localhost:8080` stalls the thread while `http-server` prepares and serves the page. Our server isn't going to give us a programmatic indication that the page is ready, so if we don't pause here, our tests would run too early and fail.

`cypress run` runs our tests in headless mode. 

## Bash Connections
### &

`&`  executes the command on the left-hand side in the background. By running `http-server &`, the server will no longer block the thread.

🤔 _There are plenty of other uses for backgrounding tasks. For example: `eslint ./src/* & npm run test`. In this case, linting and testing can be done in parallel to save some time. But that's not the whole story; we'll check in on this again a little later._

---

### &&

`&&` works similar to the same operator in JavaScript. The operation on the right-hand side only runs if the operation on the left-hand side returns `true` or in Bash, an exit status of `0`.  `wait-on` returns that `0` when the page is ready, which clears the way for `cypress run`. 

---

### ;

`;`  is the standard command separator. Unlike `&&`, `;` doesn't require a specific exit status. Our example would still work if we used `;` instead of `&&`, but that _is_ assuming everything goes as planned.

❓_If you'd like to see the consequence of `&&` vs `;` , try `npm run simulate-server-crash-with-semi`  and `npm run simulate-server-crash-with-and`  in [and-and-and-and](https://github.com/easilyBaffled/and-and-and-and)._

---

### kill

`kill`  kills a process with a given process Id.

---

### $!
`$!` is a special character that holds the process Id of the most recently backgrounded process. For us, that is our server. So when tied with `kill`, it kills the backgrounded server _after_ the tests have finished.

🤔 _There was a problem with the `eslint ./src/* & npm run test` example from before. Like with `http-server &`, there’s no way to know if `eslint` passed or failed. So we could end up with a false positive, where the tests pass but the linting fails. We can fix this with `&& wait $!`. `&&` ensures that we only check the linting results if our tests passed. `wait` is a Bash command that waits for a given process do finish and return the process’s exit code. When paired with `$!`, it will wait for the backgrounded tasks and (provided it can finish) returns the exit code._


## Wrap up 
There are NPM modules like [start-server-and-test](https://github.com/bahmutov/start-server-and-test)  and [concurrently](https://www.npmjs.com/package/concurrently) that can simplify this sort of work, so you don't need `&`, `&&`, `;`, but it's always nice to know a little more about another language. I'm still learning these things, so if you have caught something that I missed or have additional examples/usage, please leave a comment. I'd love to learn more. 

---
Thank you to all of my editors, Debbie, Ville, Justin, and Julian.
