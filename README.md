# KaTrain v1.0

This repository contains  tool for analyzing and playing go with AI feedback from KataGo.

The original idea was to give immediate feedback on the many large mistakes we make in terms of inefficient moves,
but has since grown to include a wide range of features, including:

* Review your games to find the moves that were most costly in terms of points lost.
* Play against AI and get immediate feedback on mistakes with option to retry.
* Play against a wide range of weakened versions of AI with various styles.
* Play against a stronger player and use the retry option instead of handicap stones.

## Screenshots


| Analyze games  | Play against an AI Teacher |
| ------------- | ------------- |
| ![screenshot](img/screenshot_analyze.png)  | ![screenshot](img/screenshot_play.png)  |

## Quickstart
* Right-click any button you don't understand for help.
* To analyze a game, load it using the button in the top right, or press `ctrl-L`
* To play against AI, pick an AI from the drop down a color and either 'human' or 'teach' for yourself and start playing.
    * For different board sizes, use the button with the little goban in the bottom right for a new game.
      
       
## Installation

### Quick Installation for Windows users
* See the releases tab for pre-built installers

### Installation from source for Windows users
* Download the repository by clicking the green *Clone or download* on this page and *Download zip*. Extract the contents.
* Make sure you have a python installation, I will assume Anaconda (Python 3.7), available [here](https://www.anaconda.com/distribution/#download-section). 
* Open 'Anaconda prompt' from the start menu and navigate to where you extracted the zip file using the `cd <folder>` command.
* Execute the command `pip install numpy kivy_deps.glew kivy_deps.sdl2 kivy_deps.gstreamer kivy`
* Start the app by running `python katrain.py` in the directory where you downloaded the scripts. Note that the program can be slow to initialize the first time, due to kata's gpu tuning.

### Installation for Linux/Mac users

* This assumed you have a working Python 3.6/3.7 installation as a default. If your default is python 2, use pip3/python3. Kivy currently does not have a release for Python 3.8.
* Git clone or download the repository.
* `pip install kivy numpy`
* Put your KataGo binary in the `KataGo/` directory or change the `engine.command` field in `config.json` to your KataGo v1.3.5+ binary.
    *  Compiled binaries and source code can be found [here](https://github.com/lightvector/KataGo/releases).
    * You will need to `chmod +x katago` your binary if you downloaded it.  
    * Executables for Mac are not available, so compiling from source code is required there.
* Start the app by running `python katrain.py`.  Note that the program can be slow to initialize the first time, due to KataGo's GPU tuning.

     
## Manual


### Play
Under the 'play' tab you can select who is playing black and white.
* Human is simple play with potential feedback, but without auto-undo.
* Teach will give you instant feedback, and auto-undo bad moves to give you a second chance. 
    * Settings for this mode can be found under 'Configure Teacher'
* AI will activate the AI in the dropdown next to the buttons.
    * Settings for the selected AI(s) can be found under 'Configure AIs'
 
If you do not want to see 'Points lost' or other feedback for your moves,
 set 'show last n dots' to 0 under 'Configure Teacher', and click on the words 'Points lost' to hide its value.
 
#### What are all these coloured dots?
The dots indicate how many points were lost by that move.

* The colour indicates the size of the mistake according to kata
* The size indicates if the mistake was actually punished. Going from fully punished at maximal size,
  to no actual effect on the score at minimal size. 

In short, if you are a weaker player you should mostly on large dots that are red or purple,
while stronger players can pay more attention to smaller mistakes.



#### AIs
Available AIs, with strength indicating an estimate for the default settings, are:

* **[9p+]** **Default** is full KataGo, above professional level. 
* **Balance** is KataGo occasionally making weaker moves, attempting to win by ~2 points. 
* **Jigo** is KataGo aggressively making weaker moves, attempting to win by 0.5 points.
* **[~4d]** **Policy** uses the top move from the policy network (it's 'shape sense' without reading), should be around high dan level depending on the model used.
* **[~3k]**: **P:Weighted** picks a random move weighted by the policy, as long as it's above `lower_bound`. `weaken_fac` uses `policy^(1/weaken_fac)`, increasing the chance for weaker moves.
* **[~5k]**: **P:Pick** picks `pick_n + pick_frac *  <number of legal moves>` moves at random, and play the best move among them.
   The setting `pick_override` determines the minimum value at which this process is bypassed to play the best move instead, preventing obvious blunders.
   This, along with 'Weighted' are probably the best choice for kyu players who want a chance of winning without playing the sillier bots below. Variants of this strategy include:
    * **[~3k]**: **P:Local** will pick such moves biased towards the last move with probability related to `local_stddev`.
    * **[~7k]**: **~P:Tenuki** is biased in the opposite way as P:Local, using the same setting.
    * **[~7k]**: **P:Influence** is biased towards 4th+ line moves, with every line below that dividing both the chance of considering the move and the policy value by `influence_weight`. Consider setting `pick_frac=1.0` to only affect the policy weight. 
    * **[~7k]**: **P:Territory** is biased in the opposite way, towards 1-3rd line moves, using the same setting. 
* * **[~7k]**: **P:Noise** mixes the policy with `noise_strength` Dirichlet noise. At `noise_strength=0.9` play is near-random, while `noise_strength=0.7` is still quite strong. A threshold setting is included to avoid senseless first-line moves. 

Selecting the AI as either white or black opens up the option to configure it under 'Configure AI'.

### Analysis
Keyboard shortcuts are shown with **[key]**.

* The checkboxes configure:
    * **[q]**: Child moves are shown. On by default, can turn it off to avoid obscuring other information or when wanting to guess the next move.
    * **[w]**: All dots: Show all evaluation dots instead of the last few. 
        * You can configure how many are shown with this setting off, and whether they are shown for AIs under 'Play/Configure Teacher'.
    * **[e]**: Top moves: Show the next moves KataGo considered, colored by their expected point loss. Small dots indicate high uncertainty. Hover over any of them to see the principal variation.
    * **[r]**: Show owner: Show expected ownership of each intersection.
    * **[t]**: NN Policy: Show KataGo's policy network evaluation, i.e. where it thinks the best next move is purely from the position, and in the absence of any 'reading'.

* The analysis buttons are used for:
    * **[a]**: Extra: Re-evaluate the position using more visits, usually resulting in a more accurate evaluation.
    * **[s]**: Equalize: Re-evaluate all currently shown next moves with the same visits as the current top move. Useful to increase confidence in the suggestions with high uncertainty.
    * **[d]**: Sweep: Evaluate all possible next moves. This can take a bit of time even though 'fast_visits' is used, but the result is nothing if not colourful.

    
## Keyboard and mouse shortcuts

In addition to shortcuts mentioned above, there are:

* **[Tab]**: to switch between analysis and play modes. (NB. keyboard shortcuts function regardless)
* **[~]** or **[`]** or **[p]**: Hide side panel UI and only show the board.
* **[enter]**: AI Move
* **[arrow up]** or **[z]**: Undo move. Hold shift for 10 moves at a time, or ctrl to skip to the start.
* **[arrow down]** or **[x]**: Redo move. Hold shift for 10 moves at a time, or ctrl to skip to the start.
* **[scroll up]**: Undo move. Only works when hovering the cursor over the board.
* **[scroll down]**: Redo move. Only works when hovering the cursor over the board.
* **[click on a move]**: See detailed statistics for a previous move.
* **[double-click on a move]**: Navigate directly to that point in the game.
* **[Ctrl-v]**: Load SGF from clipboard and do a 'fast' analysis.
* **[Ctrl-c]**: Save SGF to clipboard
* **[Ctrl-l]**: Load SGF from file and do a normal analysis.
* **[Ctrl-s]**: Load SGF to file
* **[Ctrl-n]**: Load SGF from clipboard
* **[space]**: Pass


## Configuration

Configuration is stored in `config.json`. Most settings are now available to edit in the program, but some cosmetic options are not.
You can use `python katrain.py your_config_file.json` to use another config file instead.

If you ever need to reset to the original settings, simply re-download the `config.json` file in this repository.

### Settings Panel
* engine settings
    * max_visits: the number of visits used in analyses and AI moves, higher is more accurate but slower.
    * max_time: maximal time in seconds for analyses, even when the target number of visits has not been reached.    
    * fast_visits: the number of visits used for certain operations with fewer visits.
    * katago: path to your KataGo executable.
    * model: path to your KataGo model file.
    * config: path to your KataGo config file.    
    * threads: number of threads to use in the KataGo analysis engine.
* game settings
    * init_size: the initial size of the board, on start-up.
    * init_komi: likewise, for komi.
* sgf settings
    * sgf_load: default path where the load SGF dialog opens.
    * sgf_save: path where SGF files are saved.    
* board_ui settings
    * eval_dot_max_size: size of coloured dots when point size is maximal, relative to stone size.
    * eval_dot_min_size: size of coloured dots when point size is minimmal 
    * ... various other minor cosmetic options.
* debug settings
    * level: determines the level of output in the console, where 0 shows no debug output, 1 shows some and 2 shows a lot.


## FAQ

* The program is slow to start!
  * The first startup of KataGo can be slow, after that it should be much faster.
* The program is running too slowly!
  *  Adjust the number of visits or maximum time allowed in the settings.
 

## Contributing

* Feedback and pull requests are both very welcome.
* For suggestions and planned improvements, see the 'issues' tab on github.
