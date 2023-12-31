# Magick Spell: TriageBot

**NOTE: In the current version of Magick, if you import spells with subspells you need to go through each "Spell" node and change the spell its pointing to. Upload all of the spells first, then change the spell they're pointing to (to the same name it currently is). Imported spells point to spell ID's from the project they were exported from, your newly imported spells get a new spell ID.**

The purpose of this bot is to watch a specific Discord Forum channel and gather some additional information from the creator of any bug report thread. Magick doesn't yet have a node for creating GitHub issues, but it will soon be added and this bot will actually create an issue if told to.

Logic flow:

- Get 1 event of type "State" - If the sender is not the thread OP, state is set to Resolved (this bot only responds to OP)
  - if none, it is the first message
    - store event "ThreadTitle" with value of the thread title
    - store event "ThreadBody" with the value of the thread body
    - Store "State" type event "GatheringInfo".
    - Thank for user for their bug submission.
    - Continue to "Gathering Information: Title" logic
  - if State = "GatheringInfo"
    - Get 1 event of type "Title"
      - If there is none:
        - Store an event of type "Title" with value "[QUERYING]"
        - Recall ThreadTitle event
        - Ask the user if the title of the thread is a suitable short description of the issue.
      - If there is one, and the value is "[QUERYING]":
        - Decide if the user response was Yes, No, or Other
          - If the response is Yes, store event of type "title" with the value of the (get from event "ThreadTitle"). Continue to description
          - If the response is No, ask the user for another title
          - If the response is Other, store event of type "title" with the value of their response. Continue to description
      - If there is one with another value, continue to description
    - Get 1 event of type "Description"
      - If there is none
        - ask the user if they'd like to add anything to the issue description
        - Store an event of type "description" with value "[QUERYING]"
      - If there is one, and the value is "[QUERYING]":
        - Decide if the user response was Yes, No, or Other
          - If the response is No, store event of type "description" with the value of the thread body (get from just event "ThreadBody"). Continue to Replication
          - if the response is Yes, ask the user for the additional information
          - If the response is Other, that means they have responded to the first question with extra information. Concatenate their response to end of the value from event "ThreadBody" and store event of type "description" with the value. Continue to Replication
      - If there is one with another value, continue to replication
    - Get 1 event of type "Replication"
      - If none, ask the user if they can provide steps to replicate the issue. Store an event of type "Replication" with value "[QUERYING]"
      - If there is one, and the value is "[QUERYING]":
        - Decide if the user response was Yes, No, or Other
          - If the response is Yes, ask the user what the steps for replication are
          - If the response is No, Store an event of type "Replication" with an empty value. Continue to System
          - If the response is Other, store an event of type "Replication" with the value from their response. Continue to System
      - If there is one with another value, continue to System
    - Get 1 event of type "System" if they can provide steps to replicate the issue
      - If none, ask the user for their operating system and browser. Store an event of type "System" with value "[QUERYING]"
      - If there is one, and the value is "[QUERYING]":
        - Decide if the user response was Yes, No, or Other
          - If the response is Yes, ask the user what their operating system/browser are
          - If the response is No, Store an event of type "System" with an empty value. store state type event with value "CreatingIssue" and Continue to System
          - If the response is Other, store an event of type "System" with the value from their response. store state type event with value "CreatingIssue" and Continue to creating issue
      - If there is one with another value, store state type event with value "CreatingIssue" and continue to creating issue
  - if State = "CreatingIssue"
    - Get 1 event of type "CreatingIssue"
      - If none, ask the user if they would like to create a Github Issue for this? Store a "CreatingIssue" type event with value ["QUERYING"]
      - If there is one, and the value is "[QUERYING]"
        - Decide if the user response was Yes, No, or Other
          - If the response is Yes, create the issue. Store a "State" type event with the value "Resolved". Respond saying you will create the issue.
          - If the response is No, store "State" type event with the value "Resolved". Respond saying you will not create an issue, thanks
          - If the response is Other, ask again if they'd like to make a github issue
      - If there is one with another value, continue to Resolved
  - if State = "Resolved"
    - Do nothing
