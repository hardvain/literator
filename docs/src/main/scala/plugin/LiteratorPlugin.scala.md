## Literator SBT Pluging

```scala
package laughedelic.literator.plugin

import sbt._
import Keys._
import laughedelic.literator.lib._

object LiteratorPlugin extends AutoPlugin {

  object autoImport {
    lazy val docsMap = settingKey[Map[File, File]]("Mapping between input source and output docs directories")
    lazy val docsOutputDirs = settingKey[Seq[File]]("Output directories for the generated documentation")
    lazy val docsAddIndex = settingKey[Boolean]("If true, index will be added to each generated file")
    lazy val docsCleanBefore = settingKey[Boolean]("If true, literator will clean up before generating files")
    lazy val generateDocs = taskKey[Unit]("Generates markdown docs from code using literator tool")
  }
  import autoImport._

  // This plugin will load automatically
  override def trigger = allRequirements

  // Default settings
  override lazy val projectSettings = Seq[Setting[_]](
    docsMap := Map(file(sourceDirectory.value.toString) -> file("docs/src/")),
    docsOutputDirs := docsMap.value.values.toSeq,
    docsAddIndex := false,
    docsCleanBefore := true,
    generateDocs := { 
      val s: TaskStreams = streams.value
      docsMap.value map { case (in, out) =>
        s.log.info("Generating documentation for " + in)

        if(docsCleanBefore.value && out.exists) {
          s.log.info("Cleaning up output directory " + out)
          Defaults.doClean(Seq(out), Seq())
        }

        val errors = in.literate(Some(out), withIndex = docsAddIndex.value)
        errors foreach { s.log.error(_) }

        if (errors.nonEmpty) sys.error("Couldn't generate documentation due to parsing errors")
        else s.log.info("Documentation is written to  " + out)
      }
    }
  )

}

```




[main/scala/lib/FileUtils.scala]: ../lib/FileUtils.scala.md
[main/scala/lib/LanguageMap.scala]: ../lib/LanguageMap.scala.md
[main/scala/lib/LiteratorParsers.scala]: ../lib/LiteratorParsers.scala.md
[main/scala/lib/package.scala]: ../lib/package.scala.md
[main/scala/plugin/LiteratorPlugin.scala]: LiteratorPlugin.scala.md
[main/scala/Readme.md]: ../Readme.md.md
[test/scala/Test.scala]: ../../../test/scala/Test.scala.md