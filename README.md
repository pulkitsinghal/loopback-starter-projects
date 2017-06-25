# loopback-starter-projects

Each starter project is in a branch of its own.

# Important to remember

* If you do more than just clone the starter project you need, for example, let's say you maintain this master repo, then:
    * Always stash your work
    * Make sure to cleanup after switching projects to avoid confusion
        * One Liner:

            ```
            git reset && \
                git clean -nxd && \
                git clean -fxd && \
            ```
        * Detailed

            ```
            # unstage files
            git reset && 

            # check what will all will be cleanedup - probably your last chance to stash
            git clean -nxd

            # either stash
            # ...

            #or go ahead and cleanup
            git clean -fxd && \
            ```
