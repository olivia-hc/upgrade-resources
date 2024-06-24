[Home](home.md) &nbsp; [Meta-Analysis](meta-analysis.md) &nbsp; [Behavioural](behavioural.md) &nbsp; [EEG](eeg.md) 
# Validating EEG Trigger Timings
*Python Code in MNE for Checking Timings of Pilot EEG Triggers*

```python 
import os
import matplotlib.pyplot as plt 
import numpy as np 
import mne 
import math

# Define a dictionary mapping trigger values to dilemma names
startTriggers = {
    17: "Bystander",
    21: "Footbridge",
    33: "Expensive Equipment",
    37: "Implied Consent",
'''
& so on for rest of triggers & dilemmas
'''
}
endTriggers = {
    48: "Crane",
    52: "Mine",
    20: "Trolley",
    36: "Truck"
}
responseTriggers = {
    2: "Yes",
    10: "No"
}
#testing only as intermediate will be categorised
intermediateTriggers = {
    1: "trigger1",
    18: "trigger2",
    4: "trigger3",
    8: "trigger4",
    16: "trigger5",
    32: "trigger6",
    64: "trigger7",
    128: "trigger8",
    5: "trigger9",
    6: "trigger10"
}


raw =  mne.io.read_raw_brainvision("C:/Users/oh298/OneDrive - University of Exeter/Desktop/test eeg/Test_0099.vhdr")

events_full = mne.events_from_annotations(raw)
print(events_full)
event = events_full[0]
print(event)
events = event[1:]
print(events)

fps = 30

# Separate the first column (timestamps) from the rest of the events array
timestamps = events[:, 0]
other_columns = events[:, 1:]

# Convert the timestamps to 30 fps
timestamps_sec = timestamps / raw.info['sfreq']
timestamps_sec *= fps

# Combine the converted timestamps with the other columns
converted_events = np.column_stack((timestamps_sec, other_columns))

# Now converted_events contains the first column converted to 30 fps while preserving the data type of the other columns
print(converted_events)

events = converted_events
print(events)

segmented_data = []

start_trigger = None
response_trigger = None
# Iterate through events
for i in range(len(events)):
    event_id = events[i][2]

    # Check if the event is a start trigger
    if event_id in startTriggers.keys():
        start_trigger = i
    
    # Check if the event is a response trigger
    elif event_id in responseTriggers.keys():
        if start_trigger is not None:
            segmented_data.append(events[start_trigger:i+1])  # Create a segment
            start_trigger = None

# If a segment is still ongoing at the end, create the last segment
if start_trigger is not None:
    segmented_data.append(events[start_trigger:])
    
# Initialize lists to store frame numbers and trigger values for each trigger type
intermediate_frames_per_segment = []
end_frames_per_segment = []
start_trigger_values_per_segment = []
intermediate_trigger_values_per_segment = []
end_trigger_values_per_segment = []

# Iterate through each segmented data
for segment in segmented_data:
    start_time = segment[0][0]  # Get the start time of the segment
    start_trigger_value = None
    
    # Initialize lists to store frame numbers and trigger values for the current segment
    intermediate_frames = []
    end_frames = []
    intermediate_trigger_values = []
    end_trigger_value = None
    
    # Iterate through each event in the segment
    for event in segment:
        event_id = event[2]  # Get the event ID
        
        # Check if the event is a start trigger
        if event_id in startTriggers.keys():
            start_trigger_value = startTriggers[event_id]
        
        # Check if the event is an intermediate trigger
        elif event_id in intermediateTriggers.keys():
            intermediate_frames.append(event[0] - start_time)  # Calculate and store frame number
            intermediate_trigger_values.append(intermediateTriggers[event_id])
        
        # Check if the event is an end trigger
        elif event_id in endTriggers.keys():
            end_frames.append(event[0] - start_time)  # Calculate and store frame number
            end_trigger_value = endTriggers[event_id]
    
    # Store the frame numbers and trigger values for the current segment
    intermediate_frames_per_segment.append(intermediate_frames)
    end_frames_per_segment.append(end_frames)
    start_trigger_values_per_segment.append(start_trigger_value)
    intermediate_trigger_values_per_segment.append(intermediate_trigger_values)
    end_trigger_values_per_segment.append(end_trigger_value)

# Print the frame numbers and trigger values for each segment
for i in range(len(segmented_data)):
    print(f"Segment {i + 1}:")
    print("Intermediate Triggers:")
    for j, frame in enumerate(intermediate_frames_per_segment[i]):
        print(f"Frame {frame}: {intermediate_trigger_values_per_segment[i][j]}")
    print("End Trigger:")
    print(f"Frame {end_frames_per_segment[i][-1]}: {end_trigger_values_per_segment[i]}")
    print("Start Trigger:")
    print(f"{start_trigger_values_per_segment[i]}\n")


# Initialize a list to store segmented data with triggers
segmented_data_with_triggers = []

# Iterate through each segmented data
for segment, start_trigger_value, intermediate_frames, intermediate_trigger_values, end_frame, end_trigger_value in zip(
    segmented_data, start_trigger_values_per_segment, intermediate_frames_per_segment, 
    intermediate_trigger_values_per_segment, end_frames_per_segment, end_trigger_values_per_segment):
    
    # Construct a dictionary for the current segment
    segment_dict = {
        "start_trigger": start_trigger_value,
        "intermediate_triggers": list(zip(intermediate_frames, intermediate_trigger_values)),
        "end_trigger": (end_frame[-1], end_trigger_value)
    }
    
    # Append the dictionary to the list
    segmented_data_with_triggers.append(segment_dict)

# Print the segmented data with triggers
for i, segment_dict in enumerate(segmented_data_with_triggers):
    print(f"Segment {i + 1}:")
    print("Start Trigger:", segment_dict["start_trigger"])
    print("Intermediate Triggers:")
    for frame, trigger_value in segment_dict["intermediate_triggers"]:
        print(f"Frame {frame}: {trigger_value}")
    print("End Trigger:", segment_dict["end_trigger"][1], "at Frame", segment_dict["end_trigger"][0])
    print()
        
movie_numbers = {
    ("Bystander", "Crane"): 1,
    ("Bystander", "Mine"): 1,
    ("Bystander", "Trolley"): 3,
    ("Bystander", "Truck"): 3,
    ("Footbridge", "Crane"): 7,
    ("Footbridge", "Mine"): 7,
    ("Footbridge", "Trolley"): 6,
    ("Footbridge", "Truck"): 6,
'''
& so on for all dilemma & context pairings w. group number
'''
}

# Iterate through each segmented data
for segment_dict in segmented_data_with_triggers:
    start_trigger = segment_dict["start_trigger"]
    end_trigger = segment_dict["end_trigger"][1]
    
    # Find the corresponding movie number for the start and end triggers
    movie_number = None
    for key, value in movie_numbers.items():
        print("Start Trigger:", start_trigger)
        print("End Trigger:", end_trigger)
        print("Key:", key)
        if start_trigger == key[0] and end_trigger == key[1]:
            movie_number = value
            break
    
    # Add the movie_number aspect to the segment dictionary
    segment_dict["movie_number"] = movie_number

# Print the updated segmented data with triggers
for i, segment_dict in enumerate(segmented_data_with_triggers):
    print(f"Segment {i + 1}:")
    print("Start Trigger:", segment_dict["start_trigger"])
    print("Intermediate Triggers:")
    for frame, trigger_value in segment_dict["intermediate_triggers"]:
        print(f"Frame {frame}: {trigger_value}")
    print("End Trigger:", segment_dict["end_trigger"][1], "at Frame", segment_dict["end_trigger"][0])
    print("Movie Number:", segment_dict["movie_number"])
    print()
    
trigger_data = {
    1: [
        (20, "trigger1"),
        (24, "trigger2"),
        (60, "trigger3"),
        (61, "trigger4"),
        (89, "trigger5"),
        (90, "trigger6")
    ],
    7: [
        (20, "trigger1"),
        (24, "trigger2"),
        (42, "trigger3"),
        (50, "trigger4"),
        (59, "trigger5"),
        (60, "trigger6"),
        (90, "trigger7"),
        (97, "trigger8")
    ],
'''
& so on for all movie numbers & their attributed triggers & trigger values 
'''
}

# Define a function to round the frame numbers up to the nearest whole number
def round_frame_up(frame):
    return math.ceil(frame)

# Create a list to store the results
results = []

# Iterate through each segment dictionary
for i, segment_dict in enumerate(segmented_data_with_triggers, start=1):
    movie_number = segment_dict["movie_number"]
    intermediate_triggers = segment_dict["intermediate_triggers"]
    
    # Check if the movie number exists in the trigger_data dictionary
    if movie_number in trigger_data:
        expected_triggers = trigger_data[movie_number]
        
        # Round the frame numbers in the intermediate triggers
        rounded_intermediate_triggers = [(round_frame_up(frame), trigger) for frame, trigger in intermediate_triggers]
        
        # Check if the number of intermediate triggers matches the expected number
        if len(rounded_intermediate_triggers) == len(expected_triggers):
            # Check if the rounded intermediate triggers match the expected triggers
            if all((frame, trigger) == expected_triggers[i] for i, (frame, trigger) in enumerate(rounded_intermediate_triggers)):
                results.append((i, True, True))
            else:
                results.append((i, False, True))  # Numbers don't match
        else:
            results.append((i, False, False))  # Number of triggers doesn't match
    else:
        results.append((i, False, False))  # Movie number not found in trigger_data

# Print the results
for result in results:
    print(f"Segment {result[0]}: Numbers matched: {result[1]}, Enough numbers: {result[2]}")

# Iterate through the results and print segments where any returned false
for result in results:
    if not all(result[1:]):  # Check if any of the results returned false
        segment_dict = segmented_data_with_triggers[result[0] - 1]
        print(f"Segment {result[0]}:")
        print("Start Trigger:", segment_dict["start_trigger"])
        print("Intermediate Triggers:")
        for frame, trigger_value in segment_dict["intermediate_triggers"]:
            print(f"Frame {frame}: {trigger_value}")
        print("End Trigger:", segment_dict["end_trigger"][1], "at Frame", segment_dict["end_trigger"][0])
        print("Movie Number:", segment_dict["movie_number"])
        print()

```
