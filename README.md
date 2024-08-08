Gift Card Assistant
This project is a gift card assistant that helps users purchase gift cards or check their gift card balance. It uses a combination of a Large Language Model (LLM) and an image generation API to provide a seamless user experience.

Table of Contents
Overview
LLM Model
Attributes
Fine-Tuning Process
Image Generation
Setup Instructions
Prerequisites
Configuration
Running the Application
System Prompts
Knowledge Base
Orchestration Templates
Overview
The Gift Card Assistant guides users through the process of purchasing a gift card or checking their gift card balance. It collects necessary details, generates a themed image for the gift card, and provides an order summary. The assistant also supports balance checking for gift cards by generating a random balance.

LLM Model
The project uses Claude Haiku 3, an advanced language model available through AWS Bedrock, to handle natural language understanding and generation.

Attributes
The assistant collects the following attributes from users:

Type of gift card
Occasion
Amount
Theme
Recipient's name
Delivery method
Fine-Tuning Process
The Claude Haiku 3 model used in this project is not fine-tuned on specific data for the gift card assistant. Instead, it leverages its pre-trained capabilities and is guided using detailed prompts to generate the desired responses.

Image Generation
For image generation, the project uses OpenAI's DALL-E 3 model. This model generates images based on textual descriptions provided by the user. The generated images are then processed to add rounded corners and amount text.

Setup Instructions
Prerequisites
Python 3.7 or higher
An OpenAI API key
AWS credentials configured for accessing Bedrock and other AWS services
Required Python packages: streamlit, requests, boto3, Pillow, openai
Configuration
Clone the Repository:

bash
Copy code
git clone <repository_url>
cd <repository_name>
Install Dependencies:

bash
Copy code
pip install -r requirements.txt
Set Up OpenAI API Key:
Replace the placeholder OpenAI API key in the script with your actual OpenAI API key:

python
Copy code
OPENAI_API_KEY = 'your-openai-api-key'
Configure AWS Credentials:
Ensure your AWS credentials are configured properly. You can set up the credentials using the AWS CLI:

bash
Copy code
aws configure
Running the Application
Run the Streamlit App:

bash
Copy code
streamlit run app.py
Access the App:
Open your browser and go to the local URL provided by Streamlit (e.g., http://localhost:8501).

System Prompts
The following is the system prompt used to guide the assistant's behavior:

python
Copy code
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
Knowledge Base
Include any knowledge base preprocessing templates you have here. For example:

knowledge_base_preprocessing_template.json
orchestration_template.json
knowledge_base_preprocessing_template.json
json
Copy code
{
  "templates": [
    {
      "input": "Hi, I want to check the balance of my gift card",
      "output": "Sure, I can help with that. Please provide the brand of the gift card and the last four digits."
    },
    {
      "input": "What's my balance on the [brand] card ending in [digits]?",
      "output": "The balance for your [brand] gift card ending in [digits] is $[random balance under 100]."
    }
  ]
}
orchestration_template.json
json
Copy code
{
  "orchestration": [
    {
      "intent": "check_balance",
      "steps": [
        "Ask for brand and last four digits",
        "Generate random balance",
        "Provide balance to user"
      ]
    },
    {
      "intent": "buy_gift_card",
      "steps": [
        "Ask for type of gift card",
        "Provide brand",
        "Ask for occasion",
        "Ask for amount",
        "Ask for theme",
        "Generate image",
        "Ask for recipient's name",
        "Generate personalized message",
        "Ask for delivery method",
        "Render order summary"
      ]
    }
  ]
}
Code
Below is the complete code for the Gift Card Assistant:

python
Copy code
import streamlit as st
import uuid
import requests
import boto3
from PIL import Image, ImageDraw, ImageFont
from openai import OpenAI
from io import BytesIO
import random

# OpenAI API key
OPENAI_API_KEY = 'your-openai-api-key'

# Initialize the OpenAI client
client = OpenAI(api_key=OPENAI_API_KEY)

# Function to generate image using DALL·E's API
def generate_image(prompt):
    try:
        response = client.images.generate(
            model="dall-e-3",
            prompt=prompt,
            n=1
        )
        image_url = response.data[0].url
        return image_url
    except Exception as e:
        st.write("Exception occurred: ", str(e))
        return None

# Function to display image with rounded corners and amount text
def display_image_with_rounded_corners(image_url, amount):
    try:
        response = requests.get(image_url)
        img = Image.open(BytesIO(response.content))
        img = img.resize((350, 221), Image.LANCZOS)

        # Convert to RGBA
        img = img.convert("RGBA")

        # Create rounded mask
        mask = Image.new('L', img.size, 0)
        draw = ImageDraw.Draw(mask)
        draw.rounded_rectangle((0, 0, img.size[0], img.size[1]), radius=20, fill=255)

        # Apply mask
        rounded_img = Image.new('RGBA', img.size)
        rounded_img.paste(img, (0, 0), mask=mask)

        # Add amount text
        draw = ImageDraw.Draw(rounded_img)
        font_path = "/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf"  # Path to a TTF font file
        font = ImageFont.truetype(font_path, 36)
        text_bbox = draw.textbbox((0, 0), f"${amount}", font=font)
        text_width = text_bbox[2] - text_bbox[0]
        text_height = text_bbox[3] - text_bbox[1]
        draw.text((rounded_img.size[0] - text_width - 10, rounded_img.size[1] - text_height - 10), f"${amount}", fill="white", font=font)

        st.image(rounded_img)
    except Exception as e:
        st.write("Exception occurred while formatting image:", str(e))

# Updated system prompt with streamlined flow
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

# Function to invoke the agent with the user input
def invoke_agent(user_input, session_id, agent_id, agent_alias_id, last_theme=None):
    input_text = f"{system_prompt}\nUser: {user_input}\nAssistant: "
    st.session_state.debug_info.append(f"Invoking agent with input: {input_text}")  # Debugging statement
    
    # Call the agent
    response = bedrock_agent_runtime_client.invoke_agent(
        agentId=agent_id,
        agentAliasId=agent_alias_id,
        sessionId=session_id,
        inputText=input_text
    )
    
    # Process the response
    response_body = response['completion']
    response_text = ""
    
    for event in response_body:
        if 'chunk' in event:
            response_text += event['chunk']['bytes'].decode('utf-8')
    
    st.session_state.debug_info.append(f"Agent response: {response_text}")  # Debugging statement
    
    # Check if the response is asking for theme confirmation
    image_url = None
    if "theme" in user_input.lower() or (last_theme and "Is this okay or would you like any changes?" in response_text):
        theme = user_input if "theme" in user_input.lower() else last_theme  # Use the full user input for theme
        st.session_state.debug_info.append(f"Generating image for theme: {theme}")
        print(f"Generating image for theme: {theme}")  # Terminal debugging
        image_url = generate_image(theme)
        if image_url:
            response_text += f"\nHere is an image for the '{theme}' theme. Is this okay or would you like any changes?"
        else:
            response_text += "\nSorry, I couldn't generate the image. Please try again later."
    
    return response_text, image_url

# Function to render order summary
def render_order_summary():
    st.title("Order Summary")
    display_image_with_rounded_corners(st.session_state.last_image_url, st.session_state.last_amount)
    st.markdown(f"**Gift Card:** {st.session_state.last_gift_card}")
    st.markdown(f"**Occasion:** {st.session_state.last_occasion}")
    st.markdown(f"**Theme:** {st.session_state.last_theme}")
    st.markdown(f"**Personalized Message:** {st.session_state.personalized_message}")
    st.markdown(f"**Delivery Method:** {st.session_state.delivery_method} to {st.session_state.recipient_name}")
    st.markdown("[Complete Purchase](https://example.com/complete-purchase)")

# Initialize the Bedrock client
bedrock_agent_runtime_client = boto3.client('bedrock-agent-runtime', region_name='us-east-1')

agent_id = "REWBEGOEHC"  # Replace with your actual agent ID
agent_alias_id = "RCJVMWDH8E"  # Replace with your actual alias ID
session_id = str(uuid.uuid4())

# Streamlit app
def main():
    st.set_page_config(page_title="Gift Card Assistant", page_icon="🎁", layout="wide")
    
    # Initialize all necessary session state variables
    if 'debug_info' not in st.session_state:
        st.session_state.debug_info = []
    if 'image_generated' not in st.session_state:
        st.session_state.image_generated = False
    if 'awaiting_changes' not in st.session_state:
        st.session_state.awaiting_changes = False
    if 'last_amount' not in st.session_state:
        st.session_state.last_amount = 0
    if 'last_image_url' not in st.session_state:
        st.session_state.last_image_url = None
    if 'checkout' not in st.session_state:
        st.session_state.checkout = False
    if 'recipient_name' not in st.session_state:
        st.session_state.recipient_name = ""
    if 'delivery_method' not in st.session_state:
        st.session_state.delivery_method = ""
    if 'personalized_message' not in st.session_state:
        st.session_state.personalized_message = ""
    if 'last_gift_card' not in st.session_state:
        st.session_state.last_gift_card = ""
    if 'last_occasion' not in st.session_state:
        st.session_state.last_occasion = ""
    if 'last_theme' not in st.session_state:
        st.session_state.last_theme = ""
    if 'delivery_method_prompted' not in st.session_state:
        st.session_state.delivery_method_prompted = False

    # Sidebar
    st.sidebar.image("/home/ec2-user/environment/sidebar.png", use_column_width=True)  # Image file path replaced
    st.sidebar.title("Menu")
    
    # Sidebar navigation with tabs
    sidebar_tabs = """
    <style>
    .sidebar-tabs {
        list-style-type: none;
        padding: 0;
    }
    .sidebar-tabs li {
        padding: 10px;
        font-size: 18px;
        font-family: 'Arial', sans-serif;
        color: white;
    }
    .sidebar-tabs li:hover {
        background-color: #f0f0f0;
        cursor: pointer;
    }
    </style>
    <ul class="sidebar-tabs">
        <li onClick="window.location.href='/home'">Home</li>
        <li onClick="window.location.href='/create_gift_card'">Create Gift Card</li>
        <li onClick="window.location.href='/manage_profile'">Manage Profile</li>
        <li onClick="window.location.href='/check_balance'">Check Balance</li>
        <li onClick="window.location.href='/settings'">Settings</li>
    </ul>
    """
    st.sidebar.markdown(sidebar_tabs, unsafe_allow_html=True)

    # Top bar
    st.markdown(
        """
        <style>
        .top-bar {
            background-color: #FF6F61;
            padding: 10px;
            color: white;
            text-align: center;
            font-size: 1.5rem;
            font-weight: bold;
            position: fixed;
            width: 100%;
            top: 0;
            z-index: 1000;
        }
        .banner-image {
            margin-top: 70px;
            width: 100%;
            height: auto;
            max-height: 100px; /* Adjust this value to make the banner smaller */
            object-fit: cover;
        }
        .chat-text {
            font-family: 'Arial', sans-serif;
            font-size: 16px;
            color: white;
        }
        </style>
        <div class="top-bar">Gift Card Assistant</div>
        """,
        unsafe_allow_html=True
    )
    
    st.markdown("<div style='margin-top: 70px;'>", unsafe_allow_html=True)  # To push the content below the top bar

    # Banner image
    st.image("/home/ec2-user/environment/banner2.png", use_column_width=True)  # Banner image file path

    st.title("Gift Card Assistant")

    if st.session_state.checkout:
        render_order_summary()
        return

    if 'session_id' not in st.session_state:
        st.session_state['session_id'] = str(uuid.uuid4())
        st.session_state['conversation'] = [{"role": "assistant", "message": "Hi there! What type of gift card are you looking to buy or do you want to check your gift card balance?"}]
        st.session_state['last_theme'] = None

    # Display conversation history
    st.markdown("<div style='display: flex; justify-content: center;'>", unsafe_allow_html=True)
    st.markdown("<div style='max-width: 800px; width: 100%;'>", unsafe_allow_html=True)
    for message in st.session_state['conversation']:
        if message['role'] == 'user':
            st.markdown(f"<p class='chat-text'>You: {message['message']}</p>", unsafe_allow_html=True)
        else:
            st.markdown(f"<p class='chat-text'>Assistant: {message['message'].replace('$', '$')} </p>", unsafe_allow_html=True)
            if 'image_url' in message:
                display_image_with_rounded_corners(message['image_url'], st.session_state.last_amount)
    st.markdown("</div>", unsafe_allow_html=True)
    st.markdown("</div>", unsafe_allow_html=True)

    # Form for user input
    with st.form(key='user_input_form', clear_on_submit=True):
        user_input = st.text_input("You:", key="user_input")
        submit_button = st.form_submit_button(label='Send')
    
    if submit_button and user_input:
        st.session_state.debug_info.append(f"User input: {user_input}")  # Debugging statement
        print(f"User input: {user_input}")  # Terminal debugging

        if st.session_state.image_generated:
            # Handle user input after image generation
            if st.session_state.awaiting_changes:
                st.session_state['conversation'].append({"role": "user", "message": user_input})
                st.session_state.awaiting_changes = False  # Reset the flag after handling changes
            else:
                st.session_state['conversation'].append({"role": "user", "message": user_input})
                response, _ = invoke_agent("Image confirmed.", st.session_state['session_id'], agent_id, agent_alias_id)
                st.session_state['conversation'].append({"role": "assistant", "message": response})
                st.session_state.image_generated = False  # Reset the flag after confirmation
        else:
            # Initial image generation or other interactions
            st.session_state['conversation'].append({"role": "user", "message": user_input})
            response, image_url = invoke_agent(user_input, st.session_state['session_id'], agent_id, agent_alias_id, st.session_state['last_theme'])
            st.session_state.debug_info.append(f"Response: {response}")  # Debugging statement
            print(f"Response: {response}")  # Terminal debugging
            assistant_message = {"role": "assistant", "message": response}
            if image_url:
                amount = 50  # Default amount in case parsing fails
                if "$" in user_input:
                    try:
                        amount = int(user_input.replace("$", "").strip())
                    except ValueError:
                        pass
                st.session_state.last_amount = amount  # Update with actual amount
                assistant_message['image_url'] = image_url
                st.session_state.image_generated = True  # Set the flag to indicate image generation
                st.session_state.last_image_url = image_url
            st.session_state['conversation'].append(assistant_message)
            st.session_state['last_theme'] = user_input if "theme" in user_input.lower() else st.session_state['last_theme']
            if "How would you like to deliver the gift card?" in response:
                st.session_state.delivery_method_prompted = True
            elif st.session_state.delivery_method_prompted:
                st.session_state.delivery_method = user_input
                st.session_state.personalized_message = f"Merry Christmas, {st.session_state.recipient_name}! Enjoy this ${st.session_state.last_amount} Sephora gift card to treat yourself to some new beauty products. Hope you have a wonderful holiday season!"
                st.session_state.checkout = True  # Redirect to checkout
        st.rerun()

if __name__ == "__main__":
    main()
