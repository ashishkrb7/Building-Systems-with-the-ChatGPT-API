# Building Systems with the ChatGPT API

Welcome to the "Building Systems with the ChatGPT API" project! This project aims to demonstrate how to leverage the power of the ChatGPT API to build interactive chat-based systems and applications.

## Project Overview

The "Building Systems with the ChatGPT API" project provides a practical guide and code examples for developers who want to integrate the ChatGPT API into their own applications. The project showcases various use cases and demonstrates how to leverage the API's capabilities to build interactive chat-based systems.

The ChatGPT API is a powerful tool that allows you to interact with the ChatGPT model in a straightforward manner. By making API calls, you can send messages to the model and receive its responses, creating dynamic and engaging conversational experiences.

This project serves as a starting point for developers interested in exploring the possibilities of the ChatGPT API. It provides clear examples, best practices, and a solid foundation for building chat-based systems that leverage the capabilities of the ChatGPT model.

## Getting Started

- Install [Miniconda](https://repo.anaconda.com/miniconda/Miniconda3-py310_23.3.1-0-Windows-x86_64.exe) from https:/.conda.io/en/latest/miniconda.html#windows-installers (for python)

- After Anaconda installation, go to search and run Anaconda Prompt and create virtual environment using following commands

    `conda create -y -n gpt python=3.11.0`

- Activate the conda environment

    `conda activate gpt`
    
- Clone the repository to your local machine. 

    `git clone https://github.com/ashishkrb7/Building-Systems-with-the-ChatGPT-API.git` 

- Go to working directory

    `cd Building-Systems-with-the-ChatGPT-API`

- Install the required dependencies using 

    `python -m pip install -r requirements.txt`

- Go to notebook folder

    `cd docs/notebooks`

- Create .env file. It should contain following information

    ```txt
    api_type = 
    api_base = 
    api_version = 
    OPENAI_API_KEY = 
    ```

## Conclusion
 
The ChatGPT API provides a powerful tool for building conversational systems that can generate human-like responses to user input. This project has demonstrated how to use the ChatGPT API to build simple and complex conversational systems, including systems that can tailor their responses to a specific personality and systems that can handle multi-turn conversations.

The possibilities for using the ChatGPT API are endless, and we hope that this project has inspired you to explore the potential of this API and to build your own conversational systems that can enhance the user experience and provide valuable services to users.