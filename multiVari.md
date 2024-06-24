[Home](home.md) &nbsp; [Meta-Analysis](meta-analysis.md) &nbsp; [Behavioural](behavioural.md) &nbsp; [EEG](eeg.md) 
# Code for Multiple & Variable Triggers
*Collated Python Code for Multiple and Variable Triggers in EEG Pilot Study*


```python 
######PsychoPy code for EEG pilot collated into one python script for ease######
    
    ####BEGIN EXPERIMENT####
    
    ###EEG Triggers###
    ##open EEG port##
    from psychopy import parallel #Import the parallel library
    port = parallel.ParallelPort(address='0xDFF8') #Change '0xDFF8' here to your parallel port address
    
    ###response key###
    ##assign participant to group corresponding to key mapping##
    if int(expInfo['participant']) % 2 == 0:
        group = 'A'
    else:
        group = 'B'
    thisExp.addData('keyOrder',group)
    
    
    ####PRACTICE TRIALS####
    
    ###response key###
    ##translate response keys at end of routine##
    if key_resp.keys:
        if key_order == 'FJ':
            if "f" in key_resp.keys:
                this_resp = 'yes'
            elif "j" in key_resp.keys:
                this_resp = 'no'
        if key_order == 'JF':
            if "j" in key_resp.keys:
                this_resp = 'yes'
            elif "f" in key_resp.keys:
                this_resp = 'no'
    else:
        this_resp = 'no response'
    
    thisExp.addData('Response', this_resp)
    
    
    ####MAIN TRIALS####
    
    ###EEG triggers###
    
    ##begin routine##
    
    #set all variables to false#
    #reset stimulus component
    main_movies_pulse_started = False
    main_movies_pulse_ended = False
    
    #reset end trigger component
    finished_trigger_pulse_started = False
    finished_trigger_pulse_ended = False
    
    #started trigger
    started_trigger_pulse_started = False
    started_trigger_pulse_ended = False
    
    #first trigger
    trigger1_pulse_started = False
    trigger1_pulse_ended = False
    
    #second trigger
    trigger2_pulse_started = False
    trigger2_pulse_ended = False
    '''
    & so on to trigger 10
    '''
    
    
    ##each Frame##
    '''
    animations were created in 30f/s, but PsychoPy will display in 60 f/s. Therefore, these have been doubled accordingly
    (and minus 1 as python starts with 0 i.e. 1=0)
    '''
    #started trigger#
    if main_movies.status == STARTED and not main_movies_pulse_started:
        win.callOnFlip(port.setData, int(started_trigger))
        main_movies_pulse_start_time = globalClock.getTime()
        main_movies_pulse_started = True
        print("Started trigger sent at time:", main_movies_pulse_start_time)
        thisExp.addData('stimulusStart', globalClock.getTime(format=str))
    
    if main_movies_pulse_started and not main_movies_pulse_ended:
        if globalClock.getTime() - main_movies_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            main_movies_pulse_ended = True
            print("Started trigger ended at time:", globalClock.getTime())
    
    #trigger1 - same for all#
    if frameN == 39 and not trigger1_pulse_started:
        win.callOnFlip(port.setData, 1)
        trigger1_pulse_start_time = globalClock.getTime()
        trigger1_pulse_started = True
        print("first trigger sent at time:", trigger1_pulse_start_time)
    
    if trigger1_pulse_started and not trigger1_pulse_ended:
        if globalClock.getTime() - trigger1_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            trigger1_pulse_ended = True
            print("First trigger ended at time:", globalClock.getTime())
    
    #trigger 2 - same for all#
    if frameN == 47 and not trigger2_pulse_started:
        win.callOnFlip(port.setData, 18)
        trigger2_pulse_start_time = globalClock.getTime()
        trigger2_pulse_started = True
        print("Trigger 2 sent at time:", trigger2_pulse_start_time)
    
    if trigger2_pulse_started and not trigger2_pulse_ended:
        if globalClock.getTime() - trigger2_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            trigger2_pulse_ended = True
            print("trigger2 ended at time:", globalClock.getTime())
    
    #trigger3#
    # frame30 trigger
    if frameN == 59 and int(movie_number) in [8, 9] and not trigger3_pulse_started:
        win.callOnFlip(port.setData, 4)
        trigger3_pulse_start_time = globalClock.getTime()
        trigger3_pulse_started = True
        print("trigger 3c sent at time:", trigger3_pulse_start_time)
    
    if trigger3_pulse_started and int(movie_number) in [8, 9] and not trigger3_pulse_ended:
        if globalClock.getTime() - trigger3_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            trigger3_pulse_ended = True
            print("trigger 3c ended at time:", globalClock.getTime())
    
    # frame40 trigger
    if frameN == 79 and int(movie_number) in [10] and not trigger3_pulse_started:
        win.callOnFlip(port.setData, 4)
        trigger3_pulse_start_time = globalClock.getTime()
        trigger3_pulse_started = True
        print("trigger 3d sent at time:", trigger3_pulse_start_time)
    
    if trigger3_pulse_started and int(movie_number) in [10] and not trigger3_pulse_ended:
        if globalClock.getTime() - trigger3_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            trigger3_pulse_ended = True
            print("trigger 3d ended at time:", globalClock.getTime())
    
    # frame42 trigger
    if frameN == 83 and int(movie_number) in [6, 7] and not trigger3_pulse_started:
        win.callOnFlip(port.setData, 4)
        trigger3_pulse_start_time = globalClock.getTime()
        trigger3_pulse_started = True
        print("trigger 3b sent at time:", trigger3_pulse_start_time)
    
    if trigger3_pulse_started and int(movie_number) in [6, 7] and not trigger3_pulse_ended:
        if globalClock.getTime() - trigger3_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            trigger3_pulse_ended = True
            print("trigger 3b ended at time:", globalClock.getTime())
    
    # frame60 trigger
    if frameN == 119 and int(movie_number) in [1, 2, 3, 4, 5, 11] and not trigger3_pulse_started:
        win.callOnFlip(port.setData, 4)
        trigger3_pulse_start_time = globalClock.getTime()
        trigger3_pulse_started = True
        print("trigger 3a sent at time:", trigger3_pulse_start_time)
    
    if trigger3_pulse_started and int(movie_number) in [1, 2, 3, 4, 5, 11] and not trigger3_pulse_ended:
        if globalClock.getTime() - trigger3_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            trigger3_pulse_ended = True
            print("trigger 3a ended at time:", globalClock.getTime())
    '''
    & so on to trigger 10
    '''
    
    #end trigger (4 seconds - 120)#
    '''
    Note on continueRoutine use:
    typically the end of stimuli duration is used to force end routine. However, as all videos are 4 seconds, 
    this would prevent us from being able to send and end trigger, particularly from being able to set the pins back to 
    0. Therefore, continueRoutine=True is used to prevent endRoutine until the next part
    of the code runs, returns the pins to 0, and then continueRoutine=False is used to end routine. 
    As the second line of code only requires 0.01 to pass, this is the only "delay" per say, that is added. 
    '''
    if frameN == 239 and not finished_trigger_pulse_started:
        win.callOnFlip(port.setData, int(end_trigger))
        finished_trigger_pulse_start_time = globalClock.getTime()
        finished_trigger_pulse_started = True
        continueRoutine = True
        print("Finished trigger sent at time:", finished_trigger_pulse_start_time)
    
    if finished_trigger_pulse_started and not finished_trigger_pulse_ended:
        if globalClock.getTime() - finished_trigger_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            finished_trigger_pulse_ended = True
            print("Finished trigger ended at time:", globalClock.getTime())
            continueRoutine = False
            print("Ended movie routine")
    
    
    
    ###response Keys###
    
    
    ##begin Routine##
    
    #set all variables to false#
    #reset stimulus component
    yes_main_key_resp_pulse_started = False
    yes_main_key_resp_pulse_ended = False
    
    #reset stimulus component
    no_main_key_resp_pulse_started = False
    no_main_key_resp_pulse_ended = False
    
    
    ##each Frame##
    '''
    Note on continueRoutine use:
    for response keys, the key-press is used as the factor to force end routine. However, similarly to above,
    this would prevent the pins from being set back to 0. Therefore, continueRoutine is used in the same way it is used 
    above for the end trigger.
    '''
    #YES - translates response-key mapping into 'yes' response trigger#
    # FJ keys
    if main_key_resp.status == STARTED and not yes_main_key_resp_pulse_started and key_order == 'FJ' and "f" in main_key_resp.keys:
        win.callOnFlip(port.setData, 2)
        yes_main_key_resp_pulse_start_time = globalClock.getTime()
        yes_main_key_resp_pulse_started = True
        print("Yes trigger started at:", yes_main_key_resp_pulse_start_time)
        continueRoutine = True
        print("Continuing response routine")
    
    # JF keys
    if main_key_resp.status == STARTED and not yes_main_key_resp_pulse_started and key_order == 'JF' and "j" in main_key_resp.keys:
        win.callOnFlip(port.setData, 2)
        yes_main_key_resp_pulse_start_time = globalClock.getTime()
        yes_main_key_resp_pulse_started = True
        print("Yes trigger started at:", yes_main_key_resp_pulse_start_time)
        continueRoutine = True
        print("Continuing response routine")
    
    #NO - translates response-key mapping into 'no' response trigger#
    # FJ keys
    if main_key_resp.status == STARTED and not no_main_key_resp_pulse_started and key_order == 'FJ' and "j" in main_key_resp.keys:
        win.callOnFlip(port.setData, 10)
        no_main_key_resp_pulse_start_time = globalClock.getTime()
        no_main_key_resp_pulse_started = True
        print("No trigger started at:", no_main_key_resp_pulse_start_time)
        continueRoutine = True
        print("Continuing response routine")
    
    # JF keys
    if main_key_resp.status == STARTED and not no_main_key_resp_pulse_started and key_order == 'JF' and "f" in main_key_resp.keys:
        win.callOnFlip(port.setData, 10)
        no_main_key_resp_pulse_start_time = globalClock.getTime()
        no_main_key_resp_pulse_started = True
        print("No trigger started at:", no_main_key_resp_pulse_start_time)
        continueRoutine = True
        print("Continuing response routine")
    
    #end routine & bring pins back down#
    # YES
    if yes_main_key_resp_pulse_started and not yes_main_key_resp_pulse_ended:
        if globalClock.getTime() - yes_main_key_resp_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            yes_main_key_resp_pulse_ended = True
            print("Yes trigger finished at:", globalClock.getTime())
            continueRoutine = False
            print("End response routine")
    
    # NO
    if no_main_key_resp_pulse_started and not no_main_key_resp_pulse_ended:
        if globalClock.getTime() - no_main_key_resp_pulse_start_time >= 0.01:
            win.callOnFlip(port.setData, 0)
            no_main_key_resp_pulse_ended = True
            print("No trigger finished at:", globalClock.getTime())
            continueRoutine = False
            print("End response routine")
    
    ##end Routine##
    #translates response-key mapping into yes/no key-response in PsychoPy behavioural output
    if main_key_resp.keys:
        if key_order == 'FJ':
            if "f" in main_key_resp.keys:
                this_main_resp = 'yes'
            elif "j" in main_key_resp.keys:
                this_main_resp = 'no'
        if key_order == 'JF':
            if "j" in main_key_resp.keys:
                this_main_resp = 'yes'
            elif "f" in main_key_resp.keys:
                this_main_resp = 'no'
    else:
        this_main_resp = 'no response'
    
    thisExp.addData('Main Response', this_main_resp)

```
