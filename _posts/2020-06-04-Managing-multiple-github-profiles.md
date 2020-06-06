
Many folks might be using same laptop while working on work and hobby projects. using `git config` on every repo to set email and username might not be feasible when you have multiple hobby projects.

In those cases, we can make use of [conditional includes](https://git-scm.com/docs/git-config#_conditional_includes) of git and it's supported from git 2.13 onwards.

<i> Gobal git config: </i>
```sh
[user]
    name = John Doe
    email = johndoe@xyz.com

[includeIf "gitdir:~/work/"]
    path = ~/work/.gitconfig
```

Add `.gitconfig` file in your `work` directory where you are keeping all the work related projects and add a different email id
```sh
[user]
    name = John Doe
    email = johndoe@example.com
```

#### Happy coding!
