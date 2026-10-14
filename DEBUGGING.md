1. Jag undersökte hasOverLap där bokningar kunde överlappa varandra.

2. Jag kunde återskapa problemet genom att starta applikationen och välja alternativ/option 3 (Create booking). Jag skapade först en bokning i rummet Eken den 2026-11-14 kl. 10:00-12:00,
och sedan en andra bokning i samma rum, samma datum, kl. 09:00–11:00.

3. T.ex detta resultat fick man om man hade 2 överlappande bokningar, dem listades bara i active bookings utan problem.

```
Booking 1045                Booking 1046
Customer: dennis1           Customer: dennis2
Room: eken                  Room: eken
Date: 2026-11-14            Date: 2026-11-14
Time: 10:00-12:00           Time: 09:00-11:00
Participants: 3             Participants: 3
```

4. Man borde ha fått ett felmeddelande som enligt testet ger "The room is already booked during the selected time."


5. Problemet med koden var dem sista 2 raderna i hasOverLap metoden.
```
booking.StartTime >= existing.StartTime && 
booking.StartTime < existing.EndTime
```

6. Originalkoden kontrollerade om StartTime låg efter eller lika med existing StartTime och om StartTime var innan existing EndTime.
Så koden kontrollerade om om den nya starttiden låg inuti den befintliga bokningen men den kontrollerade inte om den nya slut tiden låg inuti den befintliga bokningen.
Så om du hade en bokning som var 10:00-12:00 kunde du skapa en till bokning som gick 09:00-11:00 utan att det upptäcktes. Då lades båda till som active bookings även fast dem var i samma rum/lokal och att
dem överlappade varandra.

I mitt exempel är den nya starttiden 09:00 och den befintliga starttiden 10:00. Villkoret 09:00 >= 10:00 är falskt,
så hela villkoret blev falskt och bokningen släpptes igenom, trots att tiderna överlappar.

7. Jag ändrade dem två sista raderna, dem bytte jag till:
```
booking.StartTime < existing.EndTime &&
booking.EndTime > existing.StartTime
```

Så nu med rättad kod får man felmeddelandet: The room is already booked during the selected time.

8. Jag testade nya koden genom att skapa 2 överlappande möten med exakt samma namn, tider, participants som i övre exempel men med den nya koden och fick då felmeddelandet. Viktigt att använda samma
eller så lika inmatning som man kan när man felsöker. Jag testade också att köra testerna och fick grönt.

9. Testet som kontrollerar detta är: 

```
[Theory]
    [InlineData(8, 0, 10, 0)]
    [InlineData(10, 0, 12, 0)]
    [InlineData(9, 30, 10, 30)]
    [InlineData(8, 0, 12, 0)]
    public void Create_WhenBookingOverlapsExistingBooking_ReturnsFailure(
        int startHour,
        int startMinute,
        int endHour,
        int endMinute)
    {
        var existingBooking = CreateBooking(
            startTime: new TimeOnly(9, 0),
            endTime: new TimeOnly(11, 0));
        var service = new BookingService(
            new InMemoryBookingRepository(existingBooking));
        var newBooking = CreateBooking(
            startTime: new TimeOnly(startHour, startMinute),
            endTime: new TimeOnly(endHour, endMinute));

        var result = service.Create(newBooking);

        Assert.False(result.IsSuccessful);
        Assert.Equal(
            "The room is already booked during the selected time.",
            result.Message);
    }
```