+++
title = "Tame the Viper: Golang CLI Settings with Cobra & Viper"
author = ["Rod Morison"]
date = 2023-04-16T21:55:00-07:00
publishDate = 2023-04-13T00:00:00-07:00
draft = false
topics = ["Golang", "Viper", "Cobra"]
description = "A working introduction to Cobra and Viper, a CLI configuration module for Golang apps"
+++

## What are Cobra &amp; Viper? {#what-are-cobra-and-viper}

Golang comes with a [flag package](https://pkg.go.dev/flag) out of the box to parse garden variety command line flags. Many applications will want to use environment variables, particularly if they follow [Twelve-Factor App](https://12factor.net/) guidelines. The builtin [os package](https://pkg.go.dev/os) provides basic support. [GoDotEnv](https://pkg.go.dev/github.com/joho/godotenv) is a popular package with more functionality built in.

Otoh, [Cobra](https://github.com/spf13/cobra) &amp; [Viper](https://github.com/spf13/viper/) take configuration to the next level, combining CLI flags, environment, and a config file with support for most popular formats:

-   JSON, TOML, YAML, HCL, envfile and Java properties config files
    (from [What is Viper?](https://github.com/spf13/viper/#what-is-viper))

Viper's 45k "Imported by" stat shows the package's wide popularity among Go devs. And Cobra comes with a handy [Cobra Generator](https://github.com/spf13/cobra-cli) tool of it's own, to scaffold the code for new argument based CLI commands.


## The Problem with Viper... {#the-problem-with-viper-dot-dot-dot}

{{< figure src="/ox-hugo/green_and_black_snake_illustration-scopio-35afd1bf-d3e4-4cff-8e75-c3b7b83ccf9e.jpg" >}}

The first and biggest blocker to using Viper is documentation. [Viper's Github README](https://github.com/spf13/viper/blob/master/README.md) has breadcrumbs, but Viper needs a user manual. This document aspires to that.


## What's on the Menu {#what-s-on-the-menu}

We'll tackle Cobra + Viper features in three steps

1.  Command line flags
2.  Add environment vars
3.  Marshal the config into a Go struct

All of the code examples here are in this tame-the-viper repo.


## Getting Started {#getting-started}


### GitHub Repo for This Guide {#github-repo-for-this-guide}

The code used in this guide is available in the [tame-the-viper repo](https://github.com/rmorison/tame-the-viper).


### Assumptions {#assumptions}

You'll need a working Go development environment. Go's [Download and install](https://go.dev/doc/install) instructions are great. Personally, I use the [stefanmaric/g](https://github.com/stefanmaric/g) version manager to manage my Go compilers.

This walk-through was developed with `1.19.1`

```shell
~/Projects/github.com/rmorison/tame-the-viper   main $ g list

1.17
1.17.3
1.17.4
1.17.5
> 1.19.1
```


### Cobra-CLI {#cobra-cli}

We'll use the handy cobra-cli to generate code for new commands. This tool saves time and provides consistent starting structure.

```shell
go install github.com/spf13/cobra-cli@latest
```


### Go Mod Init {#go-mod-init}

If you're coding along at home, you'll want to replace the Github account name here and throughout the Go imports.

We start a project with

```shell
mkdir tame-the-viper
cd tame-the-viper
go mod init github.com/rmorison/tame-the-viper
```


## Command Line Flags {#command-line-flags}


### First Steps {#first-steps}

We're going to lay down first code with `cobra-cli init`. Let's take a look at that tool's options with

```shell
cobra-cli --help
```

which gives

```shell
Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
cobra-cli [command]

Available Commands:
add         Add a command to a Cobra Application
completion  Generate the autocompletion script for the specified shell
help        Help about any command
init        Initialize a Cobra Application

Flags:
-a, --author string    author name for copyright attribution (default "YOUR NAME")
--config string    config file (default is $HOME/.cobra.yaml)
-h, --help             help for cobra-cli
-l, --license string   name of license for the project
--viper            use Viper for configuration

Use "cobra-cli [command] --help" for more information about a command.
```

We can use command line flags to set an author and license to drop into our new project , but instead lets take advantage `cobra-cli`'s config file support. Create a `.cobra.yaml` file with

```yaml
author: Snake Charmer
license: MIT
useViper: true
```

Note the `useViper` setting. You can use Cobra without Viper, but Viper is where the real power lies. I always use them together.

Initialize your CLI project with

```shell
cobra-cli --config .cobra.yaml init
```

You should see

```shell
Using config file: .cobra.yaml
Your Cobra application is ready at
/home/rod/Projects/github.com/rmorison/tame-the-viper
```

and with `ls *`

```shell
LICENSE  go.mod  go.sum  main.go

cmd/:
root.go
```

You can run

```shell
go mod tidy
go run main.go
```

and see help text that you'll want to replace in the generated files:

```shell
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.
```

No big deal, but it gets better.


### Add a Command {#add-a-command}

Lets add our first command, `walk-dogs`. The `cobra-cli` builds a nice stub with

```shell
cobra-cli --config .cobra.yaml add walkDogs
```

and you'll see a new file, `cmd/walkDogs.go`. The `go run main.go --help` now shows that command

```shell
Available Commands:
completion  Generate the autocompletion script for the specified shell
help        Help about any command
walkDogs    A brief description of your command
```

Let's look at the code generated for that new command. The top of `cmd/walkDogs.go` starts with

```go
/*
   Copyright © 2023 Snake Charmer

   Permission is hereby granted, free of charge, to any person obtaining a copy
```

That text is gratis the `.cobra.yaml` config file we're using. Front matter like this can be further customized per [Configuring the cobra generator](https://github.com/spf13/cobra-cli#configuring-the-cobra-generator).

Further down is the spec for our command:

```go
var walkDogsCmd = &cobra.Command{
	Use:   "walkDogs",
	Short: "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("walkDogs called")
	},
}
```

One thing I typically change is the `Use:` value of the cobra command. The Cobra generator uses [camel case](https://en.wikipedia.org/wiki/Camel_case) for its command argument to support shells that have a problem with `-` in command strings. I don't write apps for those shells and the Unix shells I do implement for are fine with a command like `walk-dogs`, which I prefer for readability and aesthetics.

Easy to change:

```go
Use:   "walk-dogs",
```

and the commands are now

```shell
Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  walk-dogs   A brief description of your command
```


### Add a Command Specific Flag {#add-a-command-specific-flag}

Viper supports "global" flags, called persistent flags, that will apply to all commands (subcommands, to be precise), and command specific flags, that can only be used with that specific command. Let's add a `name` flag. Go to the `init()` function and replace the instructive comments so the code looks like

```go
func init() {
	rootCmd.AddCommand(walkDogsCmd)

	// Here you will define your flags and configuration settings.

	walkDogsCmd.Flags().String("name", "Bella", "Which dog gets this walk")
	viper.BindPFlag("name", walkDogsCmd.Flags().Lookup("name"))
}
```

The `walkDogsCmd.Flags()` line defines a command line flag, default value, and help text. `viper.BindPFlag` connects that flag to Viper's configuration map. Much more on that later.

If we `go run main.go walk-dogs --help` we see

```shell
Flags:
  -h, --help          help for walk-dogs
      --name string   Which dog gets this walk (default "Bella")
```

Back in `walkDogs.go` add a line to the `Run:` function,

```go
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println("walkDogs called")
	fmt.Println("walking", viper.GetString("name"))
},
```

Here, we're retrieving that config value from Viper. Try `go run main.go walk-dogs`

```shell
walkDogs called
walking Bella
```

then `go run main.go walk-dogs --name Max`

```shell
walkDogs called
walking Max
```

With the simple case covered, we can move on to what Cobra &amp; Viper really bring to the table.


## Environment Variables {#environment-variables}

Environment variables are a widely popular choice, both for local developer and infrastructure configurations. The [Twelve Factor App](https://12factor.net/) helped popularized this use case.

Because we chose to include Viper in our `cobra-cli` init, the generator dropped the following function into `cmd/root.go`

```go
// initConfig reads in config file and ENV variables if set.
func initConfig() {
	// cfgFile code snipped...
	viper.AutomaticEnv() // read in environment variables that match
```

We'll get the the omitted config file code [later](#structured-configs-using-a-config-file). It's the `AutomaticEnv()` we're interested in now. That binds Viper keys to environment variables, uppercased, of course. Therefore, in a `bash` shell we can type

```shell
NAME=Luna go run main.go walk-dogs
```

and get

```shell
walkDogs called
walking Luna
```

Viper has more detailed control of environment variable binding, see [Working with Environment Variables](https://github.com/spf13/viper#working-with-environment-variables). But `AutomaticEnv()` gets it done and, with a small customization handles more complex configurations, as well see next.


## Configs for Multiple Commands {#configs-for-multiple-commands}

Let's add a new `feed-cats` command.

```shell
cobra-cli --config .cobra.yaml add feedCats
```

and in `cmd/feedCats.go` change the `Use:` line

```go
Use:   "feed-cats",
```

In this case we want to feed lots of cats. Rewrite the `init()` function in `cmd/feedCats.go` to

```go
func init() {
	rootCmd.AddCommand(feedCatsCmd)

	// Here you will define your flags and configuration settings.

	feedCatsCmd.Flags().StringArray("names", []string{}, "Which cats to feed")
	viper.BindPFlag("cats.names", feedCatsCmd.Flags().Lookup("names"))
}
```

The `StringArray` flag supports a slice of strings and to feed all those cats. Note the `"cats.names"` key for this flag. That introduces structure that we'll utilize below.

Update the `Run: func` to

```go
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println("feedCats called")
	fmt.Println("feeding", viper.GetStringSlice("cats.names"))
},
```

and lets feed some cats.

```shell
go run main.go feed-cats --names Loki --names Callie
```

outputs

```shell
feedCats called
feeding [Loki Callie]
```


### Structure in Environment Variables {#structure-in-environment-variables}

If you try

```shell
NAMES=Binx go run main.go feed-cats
```

Binx doesn't get fed

```shell
feedCats called
feeding []
```

We could try `CATS.NAMES=Binx`, but that's not a valid environment variable name:

```shell
CATS.NAMES=Binx: command not found
```

Viper provides a rewrite rule to map structured keys to valid environment variables. Go back to `initConfig()` in `cmd/root.go` and above `viper.AutomaticEnv()` edit as follows

```go
replacer := strings.NewReplacer(".", "_")
viper.SetEnvKeyReplacer(replacer)
viper.AutomaticEnv() // read in environment variables that match
```

The `NewReplacer` + `SetEnvKeyReplacer()`  does what you expect, replaces the `.` with `_`. So now

```shell
CATS_NAMES=Binx,Mage go run main.go feed-cats
```

gives

```shell
feedCats called
feeding [Binx,Mage]
```

Great. Or not. Let's investigate with an indispensable tool for working with Viper.

Create a new `cmd/configTools.go` file with

```go
package cmd

import (
	"log"

	"github.com/spf13/viper"
	"gopkg.in/yaml.v2"
)

func yamlStringSettings() string {
	c := viper.AllSettings()
	bs, err := yaml.Marshal(c)
	if err != nil {
		log.Fatalf("unable to marshal config to YAML: %v", err)
	}
	return string(bs)
}
```

(Full D: comes straight off the [Marshalling to string](https://github.com/spf13/viper#marshalling-to-string) [sic] section in the Viper docs.)

Now, add another line to the `Run: func`, so we have

```go
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println("feedCats called")
	fmt.Println("feeding", viper.GetStringSlice("cats.names"))
	fmt.Printf(yamlStringSettings())
},
```

Now

```shell
CATS_NAMES=Binx,Mage go run main.go feed-cats
```

outputs

```shell
feedCats called
feeding [Binx,Mage]
cats:
  names: Binx,Mage
name: Bella
```

Compare that to the output of

```shell
go run main.go feed-cats --names Binx --names Mage
```

namely

```shell
feedCats called
feeding [Binx Mage]
cats:
  names:
  - Binx
  - Mage
name: Bella
```

See the problem? Our env var input is for one cat named `"Binx,Mage"`, not two cats. (And that's a awful name for a cat, too.) Viper doesn't assume any format beyond "plain string" for environment values. Nor should it, there's no standard and no reason for Viper to have an opinion.

But lets say we want to support simple, comma delimited text. We'll have to do that ourselves. Update `Run: func` to

```go
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println("feedCats called")
	viper.Set("cats.names", strings.Split(viper.GetString("cats.names"), ","))
	fmt.Println("feeding", viper.GetStringSlice("cats.names"))
	fmt.Printf(yamlStringSettings())
},
```

and on

```shell
CATS_NAMES=Binx,Mage go run main.go feed-cats
```

you should see those cats feed correctly

```shell
feedCats called
feeding [Binx Mage]
cats:
  names:
  - Binx
  - Mage
name: Bella
```

Since the parsing of env var values is on us, we can adopt whatever formatting convention we choose. JSON? Sure

```go
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println("feedCats called")
	jsonValues := viper.GetString("cats.names")
	var values []string
	if err := json.Unmarshal([]byte(jsonValues), &values); err != nil {
		fmt.Println(err, "->", jsonValues)
		os.Exit(1)
	}
	viper.Set("cats.names", values)
	fmt.Println("feeding", viper.GetStringSlice("cats.names"))
	fmt.Printf(yamlStringSettings())
},
```

and

```shell
CATS_NAMES='["Binx","Mage"]' go run main.go feed-cats
```

works.

But Houston, we have a problem.

```shell
go run main.go feed-cats --names Binx --names Mage
```

gives

```shell
feedCats called
unexpected end of JSON input ->
exit status 1
```

We hit this error because `jsonValues` var is an empty string. Why is that?

Well, when Viper matches a key via environment variable it only knows set the value to a string. Viper has no builtin rule to parse env values. But remember our flag definition

```go
feedCatsCmd.Flags().StringArray("names", []string{}, "Which cats to feed")
viper.BindPFlag("cats.names", feedCatsCmd.Flags().Lookup("names"))
```

With flags Viper builds a structured `cats` key of type `[]string`.

Viper `Get*` functions, when asked to lookup a key of the wrong type, will simply return the default value for the requested type. That is, `viper.GetString("cats.names")` returns `""` when `cats.names` is a `[]string` instead of a `string`. Viper also doesn't provide a direct way to know whether a key is set by default value, flag, or environment. It's possible to write code that introspects that (try Google or ChatGPT), but for now we'll apply a simple fix.

```go
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println("feedCats called")
	jsonValues := viper.GetString("cats.names")
	var values []string
	if err := json.Unmarshal([]byte(jsonValues), &values); err == nil {
		viper.Set("cats.names", values)
		fmt.Println("cat names from env")
	}
	fmt.Println("feeding", viper.GetStringSlice("cats.names"))
	fmt.Printf(yamlStringSettings())
},
```


### The Problem with Environment and Flag Configs {#the-problem-with-environment-and-flag-configs}

Environment and flag configurations work great as long as the information stays flat, or close to it. As configurations get more complex specifying these configs becomes a problem. We're faced with writing "one line configs", i.e., collapse JSON, YAML, etc. into a single line and dropping that into an env or flag spec. Escaping characters and line breaks for these is no fun and brittle. Easy to read YAML becomes gobbledygook in a single line.

Tl;dr is environment variables are fine for key/value pairs, where values are simple types, but brittle for structured configurations.


## Structured Configs: Using a Config File {#structured-configs-using-a-config-file}

Let's look at Viper's support for finger and eye friendly config files. As mentioned [above](#what-are-cobra-and-viper) plays nice with most common formats. We'll use YAML here.

Good news: the `feed-cats` command is ready to roll for a config file as is.

In the directory, where `main.go` lives, create `.tame-the-viper.yaml` with

```yaml
cats:
  names:
    - Simba
    - Kitty
```

Then, with a simple

```shell
go run main.go feed-cats --config .tame-the-viper.yaml
```

you should get

```yaml
Using config file: .tame-the-viper.yaml
feedCats called
feeding [Simba Kitty]
cats:
  names:
  - Simba
  - Kitty
name: Bella
```

Easy.


### Value Precedence {#value-precedence}

Now that we're mixing environment, flag, and config file settings, it's worth reviewing their precedence. Viper uses the following precedence order. Each item takes precedence over the item below it:

-   explicit call to Set
-   flag
-   env
-   config
-   key/value store
-   default
    (from [Why Viper?](https://github.com/spf13/viper))


### Customize initConfig {#customize-initconfig}

Let's look at the config file setup. The `cobra-cli` put it in the `cmd/root.go` file.

```go
// initConfig reads in config file and ENV variables if set.
func initConfig() {
	if cfgFile != "" {
		// Use config file from the flag.
		viper.SetConfigFile(cfgFile)
	} else {
		// Find home directory.
		home, err := os.UserHomeDir()
		cobra.CheckErr(err)

		// Search config in home directory with name ".tame-the-viper" (without extension).
		viper.AddConfigPath(home)
		viper.SetConfigType("yaml")
		viper.SetConfigName(".tame-the-viper")
	}
```

The `cfgFile` string is a package var that will get set by the `--config` flag, via

```go
rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.tame-the-viper.yaml)")
```

in the `init()` function.

Notice that if `cfgFile` is unset, the `else` branch above will look for `$HOME/.tame-the-viper.yaml` as the config. This behavior is from the code the `cobra-cli` put down, but we can change it. My preference is, rather than the homedir, default to a config file in the current working direction, typically the path where `main.go` is invoked from. (More of a "dotenv" behavior, fwiw.)

Change the `else` clause to

```go
// Find cwd directory.
cwd, err := os.Getwd()
cobra.CheckErr(err)

// Search config in home directory with name ".tame-the-viper" (without extension).
viper.AddConfigPath(cwd)
viper.SetConfigType("yaml")
viper.SetConfigName(".tame-the-viper")
```

And if you want to change from YAML to TOML, JSON, dotenv, etc., this is the spot. If what you want is a traditional dotenv setup, `"dotenv"` in `viper.SetConfigType` will get that done.

Also, update the doc string in the flag spec. In the `init()` function, update the config flag to

```go
rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $PWD/.tame-the-viper.yaml)")
```

Just like the "Don't commit .env files" practice, I always add the app's default config to `.gitignore` (assuming git)...

```go
.tame-the-viper.yaml
```


## Complex Configs {#complex-configs}

Above, we made the case that much beyond flat key/value settings, environment and flag configs are challenging. Let's tackle a more complex, structured config now that we have a config file to work with.

Use `cobra-cli` and add a new command:

```shell
cobra-cli --config .cobra.yaml add herdKittens
```

Change the `Use:` in `cmd/herdKittens.go` to

```go
Use:   "herd-kittens",
```

For this config we're going to dispense with environment and flag settings, and go straight to a config struct in Go and config file in YAML. We'll circle back on that dispensation later.

We're going to have Viper unmarshal a config from a YAML file into a Go struct. Under the import block of `cmd/herdKittens.go` add

```go
type KittenConfig struct {
	Mother string
	Father string
	Litter []struct {
		KittenName string
		FurPattern string
		Gender     string
	}
}

var kittenConfig KittenConfig
```

and below that, at the `Run: func`, change it to

```go
Run: func(cmd *cobra.Command, args []string) {
	fmt.Println("herdKittens called")
	if err := viper.UnmarshalKey("kittens", &kittenConfig); err != nil {
		log.Fatalf("unable to marshal config to YAML: %v", err)
	}
	fmt.Printf("herding the clowder %+v\n", kittenConfig)
},
```

Here `viper.UnmarshalKey` will find the top level "kittens" key in the config and write it's contents into `kittenConfig`.

```shell
go run main.go herd-kittens
```

outputs

```shell
Using config file: /home/rod/Projects/github.com/rmorison/tame-the-viper/.tame-the-viper.yaml
herdKittens called
herding the clowder {Mother:Cleo Father:Jack Litter:[{KittenName: FurPattern: Gender:Male} {KittenName: FurPattern: Gender:Female} {KittenName: FurPattern: Gender:Female}]}
```

There's a problem: `KittenName` and `FurPattern` are default valued, empty strings. `UnmarshalKey` uses the [mapstructure](https://pkg.go.dev/github.com/mitchellh/mapstructure) package, so we need to help Viper with the YAML keys. Amend the `KittenConfig` type as follows.

```go
type KittenConfig struct {
	Mother string
	Father string
	Litter []struct {
		KittenName string `mapstructure:"kitten_name"`
		FurPattern string `mapstructure:"fur_pattern"`
		Gender     string
	}
}
```

and the output is now

```shell
Using config file: /home/rod/Projects/github.com/rmorison/tame-the-viper/.tame-the-viper.yaml
herdKittens called
herding the clowder {Mother:Cleo Father:Jack Litter:[{KittenName:Oscar FurPattern:Tabby Gender:Male} {KittenName:Daisy FurPattern:Bicolor Gender:Female} {KittenName:Lola FurPattern:Colorpoint Gender:Female}]}
```

...Spot on!


## Mixing Environment and Flags with Complex Configs {#mixing-environment-and-flags-with-complex-configs}

Yes, you can do it, as we've shown for `feed-cats`.  There we used Viper's flag notation `"cats.names"` to connect to the YAML object structure. But that notation only goes so far. You might think that a flag key like `"kittens.litter.1.kitten_name"` would get us the first kitten's name in `herd-kittens`, but no, Viper doesn't support that.

Another limitation affects mixing `viper.Unmarshal` and flags. See [Binding flags as nested keys does not work #368](https://github.com/spf13/viper/issues/368) in the Viper issues list. Down that thread [this snippet](https://github.com/spf13/viper/issues/368#issuecomment-1127455123) is offered as a workaround. I've used that patch with success.

However, consider if the use case is compelling enough to warrant the added complexity. In my case, it did for a particular project. There I added

```go
func UnmarshalConfigKey(key string, out interface{}) error {
	settings := viper.AllSettings()
	return mapstructure.Decode(settings[key], out)
}
```

to the `cmd` package and call that instead of `viper.Unmarshal`.

Further travel down that rabbit hole is beyond the scope of this article.


## Wrapping up {#wrapping-up}


### Subcommands {#subcommands}

One feature we haven't covered is subcommands. Let's say we wanted the following groups of commands:

-   `go run main.go feed cats`
-   `go run main.go feed dogs`
-   `go run main.go herd cats`
-   `go run main.go herd cattle`

We would run the following commands

```shell
cobra-cli add feed
cobra-cli add herd
cobra-cli add cats create -p 'feedCmd'
cobra-cli add dogs create -p 'feedCmd'
cobra-cli add cats create -p 'herdCmd'
cobra-cli add cattle create -p 'herdCmd'
```

and proceed similarly to the above examples. The rest of this pattern is an exercise for the reader.


### Final thoughts {#final-thoughts}

Viper provides a reasonable solution CLI applications that just need traditional key/value "dotenv" support, on par with other packages. For complex configurations that push the limits of that simple model Viper brings much more to the table and can replace many lines of config crunching code.

It's fair to say Viper has wrinkles and subtleties that don't jump out of the package docs. Hopefully this article helps.

{{< figure src="/ox-hugo/portrait_of_woman_with_two_snakes_wrapped_around_her-scopio-b73eeac1-31fc-4229-9745-b2369af022fb.jpg" >}}
