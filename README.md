# split
Splits an AtomJump Messaging database into 2 or more, independent, child databases

Each individual child database can be inserted, using a script here, into an existing 
AtomJump Messaging database, which either has existing data, or it doesn't.


## Step 1: 
Admin chooses whether to split into n databases, or to trim off a certain % of the current database, and when to carry out the migration.

## Step 2:
Export script is run. This analyses the current groups, and handles a good split.

Factors to consider here: current size of the group in terms of message number, and growth rate of the group. We will likely want a manual .json file input to this split, as an option.

It posts a 1st warning message to each split group. "This group is set to be migrated at [hours:minutes:seconds GMT], in [x] hours time to a new server. During a [y] hour migration period, you will not be able to post new messages.  We apologise for any inconvenience during this process. If you wish to delay or cancel this process, please contact our admin team. [Flag Graphic] Translate"

Just prior to the migration, it posts a message to each split group. "This group is currently being migrated to a new server. We apologise for any inconvenience during this process. You should wait until you get a 'migration completed' message before posting again. This could take up to [y] hours. [Flag Graphic] Translate"

For atomjump.com, a group title is also written to a .json file for DNS redirection:
```
{
	[
		{
			"splitIP": "split0.xxx.yyy.zzz",	
			[
				"london",
				"news",
				"obama",
				"myowngroup"
			]
		},
		{
			"splitIP": "split1.xxx.yyy.zzz",	
			[
				"brunei",
				"music",
				"mexicocity",
				"myotherowngroup"
			]
		}
	]
}
```

Or perhaps a DNS style hosts file:
```
london.atomjump.com		split0.xxx.yyy.zzz
new.atomjump.com		split0.xxx.yyy.zzz
obama.atomjump.com		split0.xxx.yyy.zzz
myowngroup.atomjump.com		split0.xxx.yyy.zzz

brunei.atomjump.com		split1.xxx.yyy.zzz
music.atomjump.com		split1.xxx.yyy.zzz
mexicocity.atomjump.com		split1.xxx.yyy.zzz
myotherowngroup.atomjump.com		split1.xxx.yyy.zzz
```

It would probably make sense for the individual IP addresses to be set prior to this file being created.

For non-atomjump.com sites, the group title is encrypted, so we may need to use the encrypted value in the output .json file.

## Step 3:

For atomjump.com, DNS Entries are added for *.atomjump.com. This DNS usually takes around 30 minutes, but it can vary, and worst case is 24 hours. However, for this worst case group which is fairly rare, they will not see the "migration completed" message until the DNS has corrected, so they will still know not to post yet.


## Step 3:
Individual .sql files are created on the current server for each of the split databases.
We keep the existing records in the current database, at this stage, so that at least users can continue to read messages on the split forums.



## Step 4:
The .sql files are transferred to the migrated servers, physically via e.g. USB stick/portable hard-drive.


## Step 5:

An import script is run on the target servers, that reads the .sql file and imports the data into a 'transitional' set of database tables.

## Step 6:

The transitional set of tables will have a set of group IDs and user IDs (and potentially other fields) that don't match the new database. For each of these fields, we will add records to the new database, but also track the OLD ID > NEW ID translation. These translations will go into new translation tables, so that bulk SQL UPDATEs can be done, for speed, on the transitional tables.

## Step 7:

Another script is run that carries out the main translation. Once a record has been translated, there must be another indexed flag that sets it to having 'already been translated', so that the translated IDs are not re-translated.


## Step 8:

The main, translated, tables are then inserted into the live database on the migration target.  A new message is posted to the forum to say "This group's migration has been completed. Thank-you for your patience. You can continue to post messages here now. [Flag Graphic] Translate"

After a user check, the transitional set of tables are deleted.

## Step 9:

A further tidy-up script is run, and the messages are deleted from the original source server, to save space. [However, note that users and groups are not deleted from the source server, to keep the integrity of the database].

The output .sql files on this server are also deleted, if they haven't already been.