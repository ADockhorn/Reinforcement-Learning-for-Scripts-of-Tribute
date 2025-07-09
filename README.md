# Reinforcement Learning for Scripts of Tribute (SoT)

This is the corresponding Github repository of our paper **Training a Reinforcement Learning Agent for Tales of Tribute**.
We hereby provide our implementation of a reinforcement learning routine for the Tales of Tribute card game simulator.
For simplicity we combined our ontributions with the content of the original Scripts of Tribute repository at the time of the upload.
For this to work, we needed to slightly modify the original framework (see the section on Initial setup and modifications).

# Reference to the Corresponding Paper

@inproceedings { Lashmet25,
  author = {Sebastian Lashmet and Alexander Dockhorn},
  title = {Training a Reinforcement Learning Agent for Tales of Tribute},
  booktitle = {2025 IEEE Conference on Games (CoG)},
  year = {2025},
  pages = {1-4}
}


# Included Files

Our contributions can be found in Bots/ExternalLanguageBotsUtils/Python and includes of:
- **RLTraining/sot_rl_environment.py** maps Scripts of Tribute into an gymnasium RL environment. For this to work, we open a subprocess that loads two agents, the first being the rl_bridge.py and the second one being an opponent player. In each step of the environment, the rl_bridge sends the current state and available actions to the sot_rl_environment via a socket connection. The data is provided to a reinformcent learning agent, here PPO, to decide about the action. Once an action has been chosen it is sent to the rl_bridge which returns it to the Scripts of Tribute agent.
- **rl_bridge.py** serves as a communication bridge between "Scripts of Tribute" and a reinforcement learning environment. It forwards data from the environment to an arbitrary agent, here, a PPO reinforcement learning

The main workflow can be described by:
- sot_rl_environment opens up a subprocess of Scripts of Tribute using the rl_bridge as a python agent
- rl_bridge connects with the sot_rl_environment and sends information of the state and reward to the sot_rl_environment
- the sot_rl_environment provides the state as an observation to any RL agent
- the RL agent decides about the action and returns it to the sot_rl_environment
- the sot_rl_environment sends the action to the rl_bridge
- rl_bridge sends the action to Tales of Tribute

Further files relevant to our work:
- **map_action_to_vector.py**: maps the current action to a vector of characteristics that will be multiplied with the learned action preference vector
- **map_gamestate_to_vector.py**:  maps the current gamestate to a vector to be processed by our RL agent
- **evil_clone_bot**: implementation of a simplified self-playing strategy in which we keep an archive of previous agents to randomly play against
- **rl_gg.py**: another version of a simplified self-playing strategy in which we keep an archive of previous agents to randomly play against
- respective older versions of these scripts representing our work process

Note: some files may still include a "todo:" note. We have added those to the public repository, because the original code included personalized filenames. For each code section, we added a reference which files need to be linked.


## Initial Setup of the Tales of Tribute framework and Modifications

- Download the Tales of Tribute repository and extract it to some folder, from now on called "root"
- Install Visual Studion 2022
- Install .Net Package
- Open the root folder and its contained TalesOfTribute.sln file
- Build Solution inside Visual Studio and test GameRunner
    - The project will be built in several subfolders. The most relevant for us will be: "GameRunner/bin/Release/net7.0"
    - Open a terminal and navigate to the "GameRunner/bin/Release/net7.0" folder
    - from here, we can open the python example bot using the following command './GameRunner "cmd:python ../../../../Bots/ExternalLanguageBotsUtils/Python/example-bot.py" "cmd:python ../../../../Bots/ExternalLanguageBotsUtils/Python/example-bot.py" -n 10 -t 1'
        - The game will start and the bots will play against each other n times within t threads
        - more threads may improve performance, but I wouldn't use it for your RL-based bot to avoid conflicts
- in ExternalAIAdapapter.cs update the following method accordingly:
        public override Move Play(GameState gameState, List<Move> possibleMoves, TimeSpan remainingTime)
    {

        var obj = gameState.SerializeGameState();
        sw.WriteLine("{ \"State\":");        
        sw.WriteLine(obj.ToString());
        sw.WriteLine(", \"Actions\": [");
        //var obj2 = possibleMoves.Select(m => m.Command.ToString()).ToList();
        sw.WriteLine(string.Join(',', possibleMoves.Select(m => "\"" + m.ToString() + "\"").ToList()));
        sw.WriteLine("]}");
        sw.WriteLine(EOT);


        string botOutput;
        botOutput = sr.ReadLine();
        // Console.WriteLine(string.Join(",", possibleMoves.Select(m => m.ToString()).ToList()));

        // Console.WriteLine($"Bot response: {botOutput}");
        return possibleMoves[int.Parse(botOutput)];
        //return MapStringToMove(botOutput, gameState, possibleMoves);
    }
- Note: this breaks compatibility with the example-bot.py, but it is necessary to make the bots work with later to be implemented RL-based bots and we are going to fix this in a few seconds

