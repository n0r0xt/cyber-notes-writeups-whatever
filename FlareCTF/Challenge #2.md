# Flare CTF #2 writeup
## this is my writeup for second challenge of the Flare CTF

When visiting the link for the CTF http://jimmyswebspace.net/ I see a character sheet of a character with the following stats: STR, DEX, CON, INT, WIS, FLAG  
I assume that the way to get the flag is to modify the FLAG stat so I look at the source code to see if there's anything there that could have anything to do with how the stats' values are determined. This is what I found:
```javascript
                const ABCDEFG = 73;
                const DEFAULT_COOKIE =
                    "MmsaHRtrc3B8ZWsNDBFrc31/ZWsKBgdrc355ZWsABx1rc315ZWseABprc3x8ZWsPBQgOa3N5NA==";

                function getStatistics(encoded) {
                    try {
                        const raw = atob(encoded);
                        const bytes = [...raw].map(
                            (c) => c.charCodeAt(0) ^ ABCDEFG,
                        );
                        const json = String.fromCharCode(...bytes);
                        return JSON.parse(json);
                    } catch {
                        alert("Invader detected.");
                        return null;
                    }
                }

                function getCookie(name) {
                    return document.cookie
                        .split("; ")
                        .find((row) => row.startsWith(name + "="))
                        ?.split("=")[1];
                }

                let encodedStats = getCookie("stats");
                if (!encodedStats) {
                    document.cookie = `stats=${DEFAULT_COOKIE}; path=/; SameSite=Lax`;
                    encodedStats = DEFAULT_COOKIE;
                }

                const stats = getStatistics(encodedStats) || {
                    STR: 0,
                    DEX: 0,
                    CON: 0,
                    INT: 0,
                    WIS: 0,
                    FLAG: 0,
                };

                Object.keys(stats).forEach((stat) => {
                    const statValue = stats[stat];

                    const bar = document
                        .querySelector(`#stat-${stat}`)
                        .closest(".p-4")
                        .querySelector("div[aria-hidden]");
                    if (bar) {
                        bar.style.width = `${statValue}%`;
                        bar.style.backgroundColor = "orange";
                    }

                    const statNumber = document.getElementById(`stat-${stat}`);
                    if (statNumber) {
                        statNumber.textContent = statValue;
                    }
                });
```
The stats are determined by the stats cookie, and the logic for getting stats in JSON form from a give cookie can be seen in the Getstatistics() function. The cookie is first base64 decoded by the atob() function and is then XOR decrypted with key ABCDEFG which is set to 73. From then, the JSON is parsed and stats are set. Additionally, entering an invalid value raises the "invader detected" alert and sets all stats to 0.  
  
What's important here though is that XOR encryption and base64 encoding can be done in reverse, which means that if it's possible to go from cookies to stats, as shown by this code, it is also possible to go from stats back to cookies. To test this, I went into cyberchef and decoded the default cookie MmsaHRtrc3B8ZWsNDBFrc31/ZWsKBgdrc355ZWsABx1rc315ZWseABprc3x8ZWsPBQgOa3N5NA== by going from base64 and then XOR with key 73, which gave me the stats {"STR":95,"DEX":46,"CON":70,"INT":40,"WIS":55,"FLAG":0}  
  
I then changed the FLAG value to 100 so that stats looked like this: {"STR":95,"DEX":46,"CON":70,"INT":40,"WIS":55,"FLAG":100} and put it in cyberchef but with the operation reversed, so XOR key 73 first then base64 encode, which gave me the cookie MmsaHRtrc3B8ZWsNDBFrc31/ZWsKBgdrc355ZWsABx1rc315ZWseABprc3x8ZWsPBQgOa3N4eXk0  
  
Finally, I used my browser to replace my stats cookie with this one and refreshed the page, which gave me the flag flare{r3specc1ng_Al4ric_15_eXp3nsive}
