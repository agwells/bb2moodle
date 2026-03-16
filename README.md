# Step-by-Step guide for migrating courses from the Blackboard standard version to moodle version > 1.9.

## Step 1: Installation

You've already downloaded the source code for this tool. Older versions may also
be available here:

- https://developerck.com/wp-content/uploads/2020/02/reteach.zip
- https://github.com/adamzap/reteach
- https://github.com/developerck/bb2moodle
- https://github.com/MillCloud/bb2moodle

### Pre-requisites

1. Linux OS
2. Python **2** (not 3)
   - Python 2 is several years out of support, but I haven't yet updated this project to work with Python 3.
   - One way to install Python 2 and a supported pip version, is to use [PyEnv](https://github.com/pyenv/pyenv).
3. `pip` and `setuptools`
   - If you used `pyenv` then you already have this.

### Setup/install

`cd` into this directory and run:

```bash
 pip install
```

Or if you're going to hack on this program...

```bash
pip install --editable
```

You should now be able to invoke this tool at the CLI...

```console
> bb2moodle
Usage: bb2moodle [options] input.zip

bb course to moodle

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -o OUT_NAME, --outfile=OUT_NAME
                        a name for the output archive
  -f, --folder          tells bb2moodle to expect a folder and convert all
                        zips in it
```

#### Step 2: Run it

You just need to run this command

```bash
bb2moodle -f bb-sample.zip -o moodle-file.zip
```

#### Step 3: Import it

Just import the generated Moodle file into your Moodle site, via the "resource course" screen.

#### Step 4: DB Modifications

Some of the fields imported from Blackboard contain data that's too large to fit into the corresponding database field in Moodle. If the import fails, you'll need to track down which column is the problem, and expand its size.

For example, `mdl_question.name` is a `char(255)`, but questions can have longer names than that. So, you can address that by changing it to an unlimited-in-size `text` column.

```postgres
ALTER TABLE mdl_question
ALTER COLUMN name
TYPE text;
```

Here are the columns that are known to sometimes have problems:

1. `mdl_question.name`
2. `mdl_qtype_match_subquestions.answertext`

**And run the query:** ``ALTER TABLE `mdl_qtype_match_subquestions` CHANGE `answertext` `answertext` TEXT CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL;``

## Additional notes:

### Black Board Categorisation

For the purpose of course migration to Moodle, Blackboard can be categorised into 3 different systems, each with a different migration procedure.

- CE 4.0/CE 4.1 (rebranded WebCT) [ can use the webctimport tool with the “IMS Content Migration Utility”]
- **Standard Blackboard (5.5, 6, 7, 8, 9, 9.1)** **[ Follow the mentioned Process in this article]**
- Vista3/Vista4/Vista8 and CE 6/CE 8 (rebranded WebCT) [ backup files are encrypted so can not import that ]

> **“_The following process tested on a standard blackboard 9.1 Q4 course export file and importing that into moodle 2.7 LTS after following below steps”_**

### Objective

Migrating courses from **blackboard Standard 9.1 Q4** version to Moodle 2.7, via exporting from Blackboard and importing that into Moodle.

> The reason to say **moodle version > 1.9** is, all the versions utilise the moodle2 process to import and before moodle 2.0 (i.e moodle 1.9 and before) the process was different. However "> moodle 2.0" was supplied with an inbuilt converter, which supports but up to a limit and without any assurity.

### Solution

To achieve the target , you need to follow the below steps:

1. Set up / install this tool, `bb2moodle`
2. Run `bb2moodle` at the CLI to convert the exported Blackboard archive into a Moodle 1.9 course backup.
3. Import the Converted Moodle 1.9 file into Moodle version 2+
4. You may need to make some modifications to the Moodle DB, because some Blackboard data fields are larger than their corresponding Moodle database fields.

- (Or you can import it in moodle 1.9 and then upgrade moodle to 2.0 and export it to moodle 2 compatible import)
