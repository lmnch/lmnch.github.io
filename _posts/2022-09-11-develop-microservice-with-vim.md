---
title: 'Can I (a vim beginner) code a microservice but only use vim (vi improved) instead of an IDE to edit my source files (okay, not only vim but at least only in the terminal)?'
date: 2022-09-11
---

# Can I (a vim beginner) code a microservice but only use vim (vi improved) instead of an IDE to edit my source files (okay, not only vim but at least only in the terminal)?

I’m a fan of Linux and open-source software. Currently, I’m using Arch Linux (yes, i use arch btw) with i3 as a window manager. My shared history with vim is quite typical, I assume. It went from “Huh? What the hell?” over “Ah okay” to “That makes somehow sense”. My conclusion was that I should use it more often. That’s the current state. One day (yesterday), I found the plugin fzf-vim somewhere on the internet. It allows the execution of a fuzzy search via fzf (a fuzzy search tool for the terminal) and opens the search result as a new buffer in vim. So I said to myself: “Why to use electron bloat like vscode or IDEs like Eclipse if you always just edit files?”

# The Goal

The goal of my experiment was to create a simple catalog shop service that loads the prices of different products from a database via REST endpoints. For the database, I wanted to use a PostgreSQL database that runs inside a Docker container. For the microservice itself, I used Quarkus because it provides a very cool developer experience and you can run it easily in the terminal. Quarkus is a Java Framework that implements different standards from Jakarta EE and Eclipse Microprofile. The challenge should be: “Don’t touch any file with anything but vim!”


## The Setup

As already mentioned, for this project I used Docker and Java. So of course, I had to install Docker and Java-related stuff (a JDK, Maven, …). For vim, I installed vim-plug to manage other plugins and then fzf for vim. Additionally, I configured it to open the ‘:Files’ menu on Ctrl+P (like in vscode):

```
    call plug#begin(‘~/.vim/plugged’)
    Plug ‘junegunn/fzf.vim’
    call plug#end()

    nnoremap <silent> <C-p> :Files<CR>
```

For the project itself, I used the Quarkus Quickstart page which initializes the basic structure of the project for you and provides a zip folder with the project. Maybe that was cheating but I don’t care.


## Start-up

The Quarkus project wizard generated already some classes but of course, I didn’t want to call my entity classes “MyEntity” or the REST interface class “GreetingResource”, so it was time to yeet them away. This already turned out to be painful after the navigation through different directories was not so easy:

![Deleting a file in vim](_posts/images/2022-09-11/1.gif)

So, I was thinking there’s probably a way to navigate through folders like files. There’s none (or at least I couldn’t find it). So restructuring the example project was already kind of annoying. And the package structure of Java projects made its contribution to it. Luckily, Quarkus only generated two class files in one directory. So, I deleted them and created three directories for endpoints, controller classes, and data model. Finally, the development joy could really start!

## The Datamodel

I created the first entity class which should represent a category in the shop system:
![Product category entity class](_posts/images/2022-09-11/2.jpeg)

At this point, I realized I didn’t know where to import used classes. Hm. Yes. Who could have foreseen that? I googled “vim java” and found a page about using vim as a complete Java IDE (https://spacevim.org/use-vim-as-a-java-ide/). But to be honest, that seemed to be far over the top even if the screenshots looked quite cool. The goal has not been to find out if Spacevim can replace a traditional Java IDE like Eclipse, but to find out if vim with a file-opening-plugin suffices to develop microservices. It is questionable if Java was really a good decision. In the meanwhile I doubt it (Fucking packages!). But okay, I could just google the few classes I have to import.

And so, I created all the entity classes. Hereby, the split window feature and copy-paste were a blessing. I was sure, that it will begin to make fun as soon as all classes are created and I can use the quick-open feature that is part of the fzf plugin.
![Entity classes of the catalog service](_posts/images/3.jpeg)

At this point, I realized that I have to include Lombok as a maven dependency. So, I searched for it in the Maven Central Repository and added it. This would not have been easier in Eclipse. I don’t trust the Eclipse Maven Editor.

## The Endpoints

Yes, after creating the entities for the database, I continue with the endpoints. For some, the logical next step would be to write the controller classes that fetch the entities from the database or even start with the endpoints in order to define the interfaces. But no. First, the products in the storage, then the shop front, and in the end the forklift. When I’m thinking about it, this analogy makes not too much sense but to be realistic: you care about logistical problems in the end.

So, I created a new class file and this was the moment when I understood that I should not have thrown away the example classes. Because I had to google the Quarkus example at first in order to know which packages I need.

While writing the endpoints, I realized that spending some more time thinking about the endpoint would have been a good idea. Some refactoring was required but I decided to do that after writing the controller classes. Yeah, I should have started either with the REST endpoint architecture upfront or the classes that load the data from the database.

## The Database Controller

This part was also less spectacular than annoying. I realized that I know the Java Persistence API much less from the top of my head than I thought. So, it was basically a lot of googling and copy-pasting. And I was close to giving up when I realized I was basically done.
## The First Run

I wrote all the required code that I could think of. So now, it was time to see if all the work was successful. It was time to really find out what kind of developer I am. It was time to find out if I know what I’m doing or if I just type crap.

I entered the magic words to my terminal that will start the compiling process and launch the framework. In this moment the compiler would check without mercy every line I wrote. This was the time, the time… I lost my thread. Here’s the result.

```
./mvnw quarkus:dev
```

![Quarkus logs 1](_posts/images/4.jpeg)

Suprisingly only a few errors… Very basic Syntax errors but only a few of them. Hm. I really expected more. But okay, these are only the compiling errors. There’ll be probably a lot of runtime errors.

Well… fixing them causes more errors to become visible:

![Quarkus logs 2](_posts/images/5.jpeg)

After fixing the compilation errors, the server was finally launching. I was very excited and wanted to test the endpoints. Now after these few compilation errors, my hard work was going to pay out. Now, I was going to see if this whole operation was at least a nice try. But, then it threw IllegalArgumentExceptions and crashed.

![Quarkus logs 3](_posts/images/6.jpeg)

It seemed like path params have to be Strings and not even UUIDs. The question came to my head if it makes more sense to convert them at every endpoint or create a ParamConverter… I didn’t want to create a ParamConverter because then, I would have had to google so much Javadocs again even if it would have been the cleaner/more generic approach. But finding all this syntax and import stuff again is so much work without autocompthis error I got another:letion. So, I didn’t do it. After fixing the errors individually in each class the following logs appeared:

![Quarkus logs 4](_posts/images/7.jpeg)

For some reason, the transactional beans couldn’t be initialized. Without having read the whole error message, I assumed it was because of the missing persistence properties and the missing database. So I created a docker-compose file with a postgres database and added the required application properties:

![Quarkus logs 5](_posts/images/8.jpeg)

Yeah, well… That was not working.

![Quarkus logs 6](_posts/images/9.jpeg)

After some time, I figured out that the problem was the missing @ApplicationScoped annotation in the service classes. It was quite easy to add them. The vim macro feature allowed to easily record the typing of the import statement followed by annotating the class. It was cool to just press two buttons for executing the same steps in other files. In the next step, I had to fix my JPQL queries:

![Quarkus logs 7](_posts/images/10.jpeg)

And then, it was just working…

![Quarkus logs 8](_posts/images/11.jpeg)

So finally, I could test it. I added some data via postgres sql client and called the endpoint. And the result was this:

![Quarkus logs 9](_posts/images/12.jpeg)

Yeah, not exactly what I wanted. I had to add the Resteasy Jsonb Quarkus dependency. The result was at least a json list:

![Quarkus logs 10](_posts/images/13.jpeg)


I readded data in the database, changed the settings to update instead of drop-and-create, and restarted the server. Again the same result:

![Quarkus logs 11](_posts/images/14.jpeg)

And after some retries, I realized that I forgot to add the jsonb extension (no, idea how this happened). Then, it worked out:

![Quarkus logs 12](_posts/images/15.jpeg)

Finally! After all the years. All the endpoints are working. Best feeling in the world!


## Conclusion

Would I do it again? No. Was it a good idea? No. Was it fun? No. Was it a good activity for a rainy Saturday afternoon? Probably kind of, but there are definitely better ones. Did I learn something? Yes. vim is not an IDE and an IDE simplifies software development. What a surprising result!

The approach of using just vim could maybe work better for languages that are — and I’m trying to say it as diplomatic as possible — less strict in their syntax and folder structure (yes, I’m talking about Javascript and Python). Nevertheless, I learned to appreciate some cool features of vim. Especially the macro recording is something I definitely miss in Eclipse. In the future, I’m going to checkout IDE extensions like vrapper, to integrate the good experiences I made with vim in Eclipse
