# Gift Card Assistant

This project is a conversational assistant designed to help users with purchasing and managing gift cards. The assistant is capable of guiding users through the purchase process of gift cards, as well as checking the balance of existing gift cards.

## Features

- **Gift Card Purchase Assistance:** The assistant helps users select a gift card type, choose an occasion, set an amount, pick a design theme, enter recipient details, and select a delivery method.
- **Gift Card Balance Checking:** Users can check the balance of their gift cards by providing the brand and the last four digits of the card.

## LLM Model

The assistant leverages the Claude Haiku 3 LLM model via AWS Bedrock. This model is fine-tuned for handling step-by-step conversational flows specifically tailored to the gift card purchasing process.

### Attributes Used

The assistant uses the following attributes in the conversation flow:

1. **Gift Card Type:** The category of the gift card (e.g., Electronics, Restaurant, Retail, Sports, Beauty).
2. **Brand:** The specific brand of the gift card (e.g., Sephora, Ulta Beauty).
3. **Occasion:** The occasion for which the gift card is being purchased (e.g., Christmas, Birthday).
4. **Amount:** The monetary value to be placed on the gift card.
5. **Theme:** The design theme for the gift card (e.g., Christmas Teddy Bear).
6. **Recipient Name:** The name of the person receiving the gift card.
7. **Delivery Method:** The method of delivery (e.g., Email, Physical Mail).

### Image Generation

For generating custom gift card designs based on user-selected themes, the assistant uses OpenAI's DALL·E model. This model is specifically used to create unique images for gift card designs.

### Preprocessing and Orchestration

The assistant also uses knowledge base preprocessing templates and orchestration templates to manage the conversation flow and ensure that all necessary information is captured from the user.

## System Setup and Configuration

### Prerequisites

To run this assistant, you will need:

- Python 3.7 or higher
- Streamlit
- AWS account with Bedrock access
- OpenAI API key

### Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/your-repository.git
    cd your-repository
    ```

2. Install the required Python packages:
    ```bash
    pip install -r requirements.txt
    ```

### Configuration

1. **AWS Configuration:**

   - Ensure you have AWS credentials configured. The assistant uses AWS Bedrock for the LLM model (Claude Haiku 3).

2. **OpenAI API Key:**

   - Add your OpenAI API key in the script where indicated:
     ```python
     OPENAI_API_KEY = 'your-openai-api-key'
     ```

3. **Knowledge Base:**

   - Ensure the knowledge base preprocessing templates and orchestration templates are correctly set up. These templates guide the conversation flow and manage how user data is processed.

### Running the Application

To start the assistant, run the Streamlit application:

```bash
streamlit run app.py
