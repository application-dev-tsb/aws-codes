# AWS Glue

Some glue code

```
import com.amazonaws.services.glue.log.GlueLogger
import io.headhuntr.util.SparkSessionHelper.createSession
import io.headhuntr.candidate.CandidateLoaderJob

object EntryPoint {
  def main(sysArgs: Array[String]) {
    val logger = new GlueLogger
    logger.info("--------------------- start lols --------------------")
    sysArgs.foreach((arg) => logger.info("arg: " + arg))
    logger.info("--------------------- end lols --------------------")
    
// 2020-06-27 07:23:34,969 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - --------------------- start lols --------------------
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: --user-class
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: EntryPoint
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: --JOB_NAME
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: data-processor
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: --job-bookmark-option
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: job-bookmark-disable
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: param3
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: value3
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: --TempDir
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: s3://io.headhuntr.staging.data-processor/temp/glue-workspace
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: param1
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: value1
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: param2
// 2020-06-27 07:23:34,970 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: value2
// 2020-06-27 07:23:34,971 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: --JOB_ID
// 2020-06-27 07:23:34,971 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: j_c3a4cf649f48fb2943cfbd8e463a1adb26e2a20f7b25a4a89caa1fe922df280c
// 2020-06-27 07:23:34,971 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: --JOB_RUN_ID
// 2020-06-27 07:23:34,971 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - arg: jr_a20b9c4f55493de1608a57a1d6bea6160d659d8f87f7c64e09be28ab20e680c1
// 2020-06-27 07:23:34,971 INFO  [Driver] log.GlueLogger (GlueLogger.scala:info(8)) - --------------------- end lols --------------------
      
    val spark = createSession(sysArgs)
    val buildFirst = true
    val batchSize = 1000000
    val idStart = 0
    val idEnd = Long.MaxValue
    val esHost = "https://xxxx.us-east-2.es.amazonaws.com:443"
    val esIndex = "candidates"
    val dmzDir = "s3://xxx/data"
    val workingDir = "s3://xxx/temp/workspace"

    val candidateProfileList = spark.read.parquet(s"$dmzDir/cand_profile_na")
    val education = spark.read.parquet(s"$dmzDir/cand_educ_na")
    val departmentExperience = spark.read.parquet(s"$dmzDir/cand_exp_dept_hh")
    val productExperience = spark.read.parquet(s"$dmzDir/cand_exp_prod_hh")
    val seniority = spark.read.parquet(s"$dmzDir/cand_exp_sen_hh")
    val industryExperience = spark.read.parquet(s"$dmzDir/cand_exp_ind_hh")
    val certifications = spark.read.parquet(s"$dmzDir/cand_exp_cert_hh")
    val jobHistory = spark.read.parquet(s"$dmzDir/job_hist_na")
    val experienceDetails = spark.read.parquet(s"$dmzDir/cand_experience_details")

    CandidateLoaderJob(
      candidateProfileList,
      seniority,
      education,
      departmentExperience,
      productExperience,
      industryExperience,
      experienceDetails,
      jobHistory,
      certifications,
      esHost,
      esIndex,
      batchSize,
      workingDir,
      idStart,
      idEnd,
      buildFirst
    ).executeJob()
  }
}
```
