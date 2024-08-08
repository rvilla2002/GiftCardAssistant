# Gift Card Assistant

## Overview

The Gift Card Assistant is a user-friendly application that helps users purchase gift cards or check their gift card balance. The assistant leverages a combination of a Large Language Model (LLM) and an image generation API to provide a seamless user experience. It collects necessary details, generates a themed image for the gift card, and provides an order summary. Additionally, it supports balance checking for gift cards by simulating a random balance.

## Table of Contents

- [Overview](#overview)
- [LLM Model](#llm-model)
- [Attributes](#attributes)
- [Fine-Tuning Process](#fine-tuning-process)
- [Image Generation](#image-generation)
- [Setup Instructions](#setup-instructions)
  - [Prerequisites](#prerequisites)
  - [Configuration](#configuration)
  - [Running the Application](#running-the-application)
- [System Prompts](#system-prompts)
- [Knowledge Base and Orchestration Templates](#knowledge-base-and-orchestration-templates)
- [AWS Bedrock Setup](#aws-bedrock-setup)
- [Code](#code)
- [License](#license)

## LLM Model

This project uses Claude Haiku 3, an advanced language model available through AWS Bedrock, to handle natural language understanding and generation. The model is pre-trained and is not fine-tuned on specific data for this assistant.

## Attributes

The Gift Card Assistant collects the following attributes from users:
- **Type of gift card**
- **Occasion**
- **Amount**
- **Theme**
- **Recipient's name**
- **Delivery method**

## Fine-Tuning Process

The Claude Haiku 3 model is not fine-tuned for this project. Instead, it operates using detailed prompts to generate the desired responses.

## Image Generation

The project uses OpenAI's DALL-E 3 model to generate images based on user-provided textual descriptions. The generated images are processed to add rounded corners and display the gift card amount.

## Setup Instructions

### Prerequisites

- Python 3.7 or higher
- OpenAI API key
- AWS credentials configured for accessing Bedrock and other AWS services
- Required Python packages: `streamlit`, `requests`, `boto3`, `Pillow`, `openai`

### Configuration

1. **Clone the Repository:**

    ```bash
    git clone <repository_url>
    cd <repository_name>
    ```

2. **Install Dependencies:**

    ```bash
    pip install -r requirements.txt
    ```

3. **Set Up OpenAI API Key:**

    Replace the placeholder OpenAI API key in the script with your actual OpenAI API key:

    ```python
    OPENAI_API_KEY = 'your-openai-api-key'
    ```

4. **Configure AWS Credentials:**

    Ensure your AWS credentials are configured properly. You can set up the credentials using the AWS CLI:

    ```bash
    aws configure
    ```

### Running the Application

1. **Run the Streamlit App:**

    ```bash
    streamlit run app.py
    ```

2. **Access the App:**

    Open your browser and go to the local URL provided by Streamlit (e.g., `http://localhost:8501`).

## System Prompts

The system prompt is designed to guide the assistant's behavior in helping users with gift card-related queries. The assistant collects all necessary details to complete the purchase or check the gift card balance and ensures a natural and smooth conversation flow.

```python
system_prompt = """
  <Context>
  You are an intelligent assistant designed to help users with gift card-related queries. Your job is to guide the user through the gift card purchasing process step-by-step, prompting them for the necessary attributes to complete the purchase while ensuring they are aware of the available options. Additionally, you should be able to help users check the balance of their gift cards. Make the conversation natural, and if multiple preferences are provided in one message, collect them and ask what's left. Always provide clear and concise information. Never answer with more than one line and only ask one question at a time.

  You do not need to call any functions until the final step to generate the purchase link. Focus on collecting all the required information from the user.

  The program is going to start the conversation with: "Hi there! What type of gift card are you interested in? (Electronics, Restaurant, Retail, Sports, Beauty) or do you want to check your gift card balance?" so just be ready to accept a response and move from there.
  
  You are not needing to make any function calls so do not mention anything about that.

  ### Initial Greeting and Type
    - If the user wants to check their gift card balance, ask for the brand and the last four digits of the card. E.g., "Sure, I can help with that. Please provide the brand of the gift card and the last four digits."
    - Simulate a random balance check. E.g., "The balance for your [brand] gift card ending in [last four digits] is $[random balance under 100]."
    - Once the user provides a type of gift card, please look for brands in that category and offer one.
    - If the user provides the type, acknowledge it and provide a random brand from the corresponding category, then ask for the occasion. E.g., "Great choice! How about a [random brand] gift card? What occasion is the gift card for?"

  ### Occasion
  - **Occasion Response:**
    - If the user provides the occasion, acknowledge it and then ask for the amount. E.g., "Cool, a [occasion] gift card! How much would you like to put on the gift card?"

  ### Amount
  - **Amount Response:**
    - Wait for the user to enter an amount, and then proceed to asking what theme works. E.g., "Great, a $[amount] gift card."

  ### Theme
  - **Theme Response:**
    - When the user provides the theme, acknowledge it and generate an image. E.g., "Got it, '[theme]' theme. Here is an image for this theme. Is this okay or would you like any changes?"
    - If they want a change, regenerate the image with the new theme.
    - If they like the image, confirm it and proceed to ask for the recipient's name.

  ### Recipient's Name
  - **Recipient's Name Response:**
    - If the user provides the recipient's name, acknowledge it and automatically generate a short message based on the occasion and recipient's name. E.g., "Could you please provide the recipient's name?"
    - "Thank you! I've generated a personalized message for you."  Do not show the message here. just say you have generated it. No need to ask for confirmation just go to the delivery method question. 

  ### Delivery Method
  - **Delivery Method Response:**
    - If the user specifies the delivery method, acknowledge it and directly call the render_order_summary function. E.g., "How would you like to deliver the gift card? (Email or Physical Mail)"
    - When the user says the delivery method like email, you call the render_order_summary function.
  Ensure that you follow these steps meticulously, capturing all necessary details and guiding the user smoothly through the process without asking for the same information twice.
</Context>
"""
