# Contribution Guide
## Reporting Issues
* Have a look through the [Botania FAQ](https://botaniamod.net/faq.php), to see if your
  issue has been solved already
* Issues regarding "Bukkit+Forge" servers that are not reproducible with Forge only are
  **not accepted**
* Issues regarding outdated versions of the mod, especially for older versions of
Minecraft, are **not accepted**
* Unless you manually update every mod on the pack to the latest, issues regarding any
public modpack that is not officially maintained or supported by the Botania developers
(the ones in the modpack section of the website are not) are **not accepted**
* Duplicate issues or issues that have been solved already (use the search feature!) will
  be closed without warning.
* Do not tag your issues' names. "Something Broke" is prefered to "[Bug] Something Broke"
  because there's a proper label system in place.
* Suggestions are **not** accepted. In general, new features are added to Botania on
  a must-have basis. The bar for new features is quite high.

## Submitting Changes
* Run the `checkSyntax` Gradle task to make sure your changes pass our style guidelines
* The `spotlessJavaApply` Gradle task can fix most violations for you.
* **Keep PR's small**. The smaller it is, the faster we will review it. Only fix one thing
  per PR instead of piling everything into one massive PR.
* If you fix a gameplay bug or add a new gameplay feature, consider adding a GameTest for
  it (see below)
* If you hate GitHub's PR model like some of the maintainers do, feel free to email patches
  to `~williewillus/violet-moon@lists.sr.ht`. See `git-send-email.io` to set up your
  Git environment for mailing patches.

## Making a Release
1. Pull from remote, test all changes, and commit everything.
2. `git tag -a release-<VERSION>`. All Botania versions *must* follow the version format
   `<MC-VER>-INT`, so it'll probably look like `git tag -a release-1.16.3-407`.
3. In the Git editor that pops up, write the changelog. Finish the tag process (usually by
   saving and closing the editor).
4. Copy the changelog to the webpage version under `web/changelog.txt`.`
5. Run `./gradlew incrementBuildNumber --no-daemon` to increment the build number of the
   next release. Commit this and the changelog.
6. Push: `git push origin master --tags`
7. Go to [Jenkins](https://ci.blamejared.com/job/Botania/view/tags/) and wait for the tag
   you just pushed to be compiled and built
8. Download the JAR and submit it to CurseForge
9. Push the website: `./syncweb.sh <remote username>`. If you don't provide a remote
   username to ssh into the webserver, it'll take your current login name.

## Working with GameTest
1. Create a structure if wanted:
   1. Run /test create <size> to generate a test platform of the desired size. IMPORTANT:
      All tests should size themselves appropriately to ensure they don't interfere with
      other tests. E.g. a test testing a block with max radius 8 should leave at least 8
      blocks of room on all sides.
   2. Build the test setup then save the structure, *without a namespace*. Then run `/test
      export <thename>`.
   3. The command will print the location of the saved `.snbt` file in
      `run/gameteststructures`.
   4. Copy the file to `src/main/resources/data/botania/gametest/structures`.
2. Create a class in `src/main/java/vazkii/botania/test`. Fill it with methods annotated
   with `@GameTest`, see the other tests for examples.
3. List the class in the `fabric-gametest` entrypoint in Botania's `fabric.mod.json`.

Tips:
* The @GameTest annotation takes a `batch` argument. Any tests in the same batch run in
  parallel. Tests in different batches will not run together.
* Please keep 5 blocks of padding in the N/S/E/W directions of all tests that have mana
  pools, spreaders, or flowers in them. This prevents them interfering with the tests
  about seeing whether flowers bind to the closest spreader, if Gametest happens to put
  them next to each other.
* If your test has a *lot* of air blocks in it, use structure voids to keep the filesize
  down. If you forget, run a regex find-and-replace on the `snbt` file, replacing
  `.*minecraft:air.*\n` with the empty string.

To run tests in-game: open the game normally, make sure you're not standing near anything
you care about, and use `/test runall`. (Gametest uses a default superflat with Generate
Structures, doMobSpawning, and doWeatherCycle all disabled as its testing environment.)

To run tests headlessly (all of these do the same thing):
* Use the `Minecraft Game Test` run configuration in your IDE.
* Launch the server with the argument `-Dfabric-api.gametest=1`.
* Use `./gradlew runGameTest`. (Github Actions does this)

After running tests headlessly, the testing world will be saved to `run/world`. Copy this
directory into `run/saves` and it will appear as a save file in singleplayer.