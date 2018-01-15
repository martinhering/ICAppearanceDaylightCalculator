# How Instacast does Night-Mode

Night-Mode or Dark-Mode has been a favourite feature to request by users for all kinds of apps. The reason why is obvious. You use your phone in all kinds of situations day and night, and during the night your eyes are accustomed to less light. Looking at a glaring smartphone display can be very jarring and you might have to turn down the display brightness. But that’s not a good solution. Everything on the text just becomes more difficult to make out. With a Night-Mode or Dark-Mode feature however the color of the text and the background sort of become inverted and it’s much easier on the eye at night.

Since there is no dedicated night-mode setting in iOS (Android may have that), app makers reside to all kinds of techniques to switch the mode between day and night setting automatically. A method I often see is using the ambient light sensor. This method however can be very annoying in situations where it’s not light nor dark and often apps switch back and forth multiple times. When I added this feature to Instacast, I wanted to go a different route. I wanted Instacast to switch only 2 times a day, in the morning and in the evening. So, I came up with what I think was a unique method at the time.

Instacast is not measuring the actual light in the room. Instead it asks for your broad location data and depending on your actual location on the planet, it calculates when the sun is setting or rising approximately. Now you might say, that querying the location and triggering GPS and thus draining the battery only for switching user interface colours is wasteful, and you’re right. That’s why Instacast is only asking for your approximate location. This location is already stored by the phone when it is connected to a cell tower, so only already stored data is passed to the app. This way no actual sensor is fired at that time. After all for the calculation of the sun angle, it doesn’t really matter if your 100 kilometres here or there. Also, please don’t look at the sun and wait for Instacast to switch the night mode setting. The actual calculation is only an approximation. It does not account for terrain height or other physical circumstances, like the earth not really being ball, but a potato.

**Usage:**

    - (BOOL) _updateSwitchSchedule
    {
        BOOL isNight = NO;
    
        ICAppearanceDaylightCalculator* calculator = [[ICAppearanceDaylightCalculator alloc] initWithWithLocation:self.location date:[NSDate date]];
        NSTimeInterval sunriseInterval = [[NSDate date] timeIntervalSinceDate:calculator.sunrise];
        NSTimeInterval sunsetInterval = [[NSDate date] timeIntervalSinceDate:calculator.sunset];
	
        if ( sunriseInterval < 0 && sunsetInterval < 0 ) {
            isNight = YES;
            self.nextSwitchDate = calculator.sunrise;
        }
        else if (sunriseInterval >= 0 && sunsetInterval < 0) {
            isNight = NO;
            self.nextSwitchDate = calculator.sunset;
        }
        else if (sunriseInterval >= 0 && sunsetInterval >= 0) {
            calculator.date = [NSDate dateWithTimeIntervalSinceNow:86400];
            NSTimeInterval tomorrowSunriseInterval = [[NSDate date] timeIntervalSinceDate:calculator.sunrise];
            NSTimeInterval tomorrowSunsetInterval = [[NSDate date] timeIntervalSinceDate:calculator.sunset];
            if ( tomorrowSunriseInterval < 0 && tomorrowSunsetInterval < 0 ) {
                isNight = YES;
                self.nextSwitchDate = calculator.sunrise;
            }
                else if (tomorrowSunriseInterval >= 0 && tomorrowSunsetInterval < 0) {
                isNight = NO;
                self.nextSwitchDate = calculator.sunset;
            }
         }
    
        //DebugLog(@"next appearance switch date: %@", [self.nextSwitchDate descriptionWithLocale:[NSLocale currentLocale]]);
        return isNight;
    }
