---
title: Backtracking in regular expressions
description: Backtracking in regular expressions
keywords: .NET, .NET Core
author: dotnet-bot
ms.author: dotnetcontent
ms.date: 07/28/2016
ms.topic: article
ms.prod: .net
ms.technology: dotnet-standard
ms.devlang: dotnet
ms.assetid: 8a3e6298-26b7-4c99-bd97-c9892f6c9418
---

# Backtracking in regular expressions

Backtracking occurs when a regular expression pattern contains optional [quantifiers](quantifiers.md) or [alternation constructs](alternation.md), and the regular expression engine returns to a previous saved state to continue its search for a match. Backtracking is central to the power of regular expressions; it makes it possible for expressions to be powerful and flexible, and to match very complex patterns. At the same time, this power comes at a cost. Backtracking is often the single most important factor that affects the performance of the regular expression engine. Fortunately, the developer has control over the behavior of the regular expression engine and how it uses backtracking. This topic explains how backtracking works and how it can be controlled.

> [!NOTE]
> In general, a Nondeterministic Finite Automaton (NFA) engine like the  regular expression engine places the responsibility for crafting efficient, fast regular expressions on the developer.
 
This topic contains the following sections:

* [Linear comparison without backtracking](#linear-comparison-without-backtracking)

* [Backtracking with optional quantifiers or alternation constructs](#backtracking-with-optional-quantifiers-or-alternation-constructs)

* [Backtracking with nested optional quantifiers](#backtracking-with-nested-optional-quantifiers)

* [Controlling backtracking](#controlling-backtracking)

## Linear comparison without backtracking

If a regular expression pattern has no optional quantifiers or alternation constructs, the regular expression engine executes in linear time. That is, after the regular expression engine matches the first language element in the pattern with text in the input string, it tries to match the next language element in the pattern with the next character or group of characters in the input string. This continues until the match either succeeds or fails. In either case, the regular expression engine advances by one character at a time in the input string.

The following example provides an illustration. The regular expression `e{2}\w\b` looks for two occurrences of the letter "e" followed by any word character followed by a word boundary.

```csharp
using System;
using System.Text.RegularExpressions;

public class Example
{
   public static void Main()
   {
      string input = "needing a reed";
      string pattern = @"e{2}\w\b";
      foreach (Match match in Regex.Matches(input, pattern))
         Console.WriteLine("{0} found at position {1}", 
                           match.Value, match.Index);
   }
}
// The example displays the following output:
//       eed found at position 11
```

```vb
Imports System.Text.RegularExpressions

Module Example
   Public Sub Main()
      Dim input As String = "needing a reed"
      Dim pattern As String = "e{2}\w\b"
      For Each match As Match In Regex.Matches(input, pattern)
         Console.WriteLine("{0} found at position {1}", _
                           match.Value, match.Index)
      Next   
   End Sub
End Module
' The example displays the following output:
'       eed found at position 11
```

Although this regular expression includes the quantifier `{2}`, it is evaluated in a linear manner. The regular expression engine does not backtrack because `{2}` is not an optional quantifier; it specifies an exact number and not a variable number of times that the previous subexpression must match. As a result, the regular expression engine tries to match the regular expression pattern with the input string as shown in the following table.

Operation | Position in pattern | Position in string | Result
--------- | ------------------- | ------------------ | ------ 
1 | e | "needing a reed" (index 0) | No match. 
2 | e | "eeding a reed" (index 1) | Possible match. 
3 | e{2} | "eding a reed" (index 2) | Possible match. 
4 | \w | "ding a reed" (index 3) | Possible match.
5 | \b | "ing a reed" (index 4) | Possible match fails. 
6 | e | "eding a reed" (index 2) | Possible match. 
7 | e{2} | "ding a reed" (index 3) | Possible match fails. 
8 | e | "ding a reed" (index 3) | Match fails. 
9 | e | "ing a reed" (index 4) | No match.
10 | e | "ng a reed" (index 5) | No match.
11 | e | "g a reed" (index 6) | No match.
12 | e | " a reed" (index 7) | No match.
13 | e | "a reed" (index 8) | No match.
14 | e | " reed" (index 9) | No match.
15 | e | "reed" (index 10) | No match
16 | e | "eed" (index 11) | Possible match.
17 | e{2} | "ed" (index 12) | Possible match.
18 | \w | "d" (index 13) | Possible match.
19 | \b | "" (index 14) | Match.
 

If a regular expression pattern includes no optional quantifiers or alternation constructs, the maximum number of comparisons required to match the regular expression pattern with the input string is roughly equivalent to the number of characters in the input string. In this case, the regular expression engine uses 19 comparisons to identify possible matches in this 13-character string. In other words, the regular expression engine runs in near-linear time if it contains no optional quantifiers or alternation constructs.

## Backtracking with optional quantifiers or alternation constructs

When a regular expression includes optional quantifiers or alternation constructs, the evaluation of the input string is no longer linear. Pattern matching with an NFA engine is driven by the language elements in the regular expression and not by the characters to be matched in the input string. Therefore, the regular expression engine tries to fully match optional or alternative subexpressions. When it advances to the next language element in the subexpression and the match is unsuccessful, the regular expression engine can abandon a portion of its successful match and return to an earlier saved state in the interest of matching the regular expression as a whole with the input string. This process of returning to a previous saved state to find a match is known as backtracking.

For example, consider the regular expression pattern `.*(es)`, which matches the characters "es" and all the characters that precede it. As the following example shows, if the input string is "Essential services are provided by regular expressions.", the pattern matches the whole string up to and including the "es" in "expressions".

```csharp
using System;
using System.Text.RegularExpressions;

public class Example
{
   public static void Main()
   {
      string input = "Essential services are provided by regular expressions.";
      string pattern = ".*(es)"; 
      Match m = Regex.Match(input, pattern, RegexOptions.IgnoreCase);
      if (m.Success) {
         Console.WriteLine("'{0}' found at position {1}", 
                           m.Value, m.Index);
         Console.WriteLine("'es' found at position {0}", 
                           m.Groups[1].Index);      
      } 
   }
}
//    'Essential services are provided by regular expres' found at position 0
//    'es' found at position 47
```

```vb
Imports System.Text.RegularExpressions

Module Example
   Public Sub Main()
      Dim input As String = "Essential services are provided by regular expressions."
      Dim pattern As String = ".*(es)" 
      Dim m As Match = Regex.Match(input, pattern, RegexOptions.IgnoreCase)
      If m.Success Then
         Console.WriteLine("'{0}' found at position {1}", _
                           m.Value, m.Index)
         Console.WriteLine("'es' found at position {0}", _
                           m.Groups(1).Index)                  
      End If     
   End Sub
End Module
'    'Essential services are provided by regular expres' found at position 0
'    'es' found at position 47
```

To do this, the regular expression engine uses backtracking as follows:

* It matches the `.*` (which matches zero, one, or more occurrences of any character) with the whole input string.

* It attempts to match "e" in the regular expression pattern. However, the input string has no remaining characters available to match.

* It backtracks to its last successful match, "Essential services are provided by regular expressions", and attempts to match "e" with the period at the end of the sentence. The match fails.

* It continues to backtrack to a previous successful match one character at a time until the tentatively matched substring is "Essential services are provided by regular expr". It then compares the "e" in the pattern to the second "e" in "expressions" and finds a match.

* It compares "s" in the pattern to the "s" that follows the matched "e" character (the first "s" in "expressions"). The match is successful.

When you use backtracking, matching the regular expression pattern with the input string, which is 55 characters long, requires 67 comparison operations. Interestingly, if the regular expression pattern included a lazy quantifier, `.*?(es),` matching the regular expression would require additional comparisons. In this case, instead of having to backtrack from the end of the string to the "r" in "expressions", the regular expression engine would have to backtrack all the way to the beginning of the string to match "Es" and would require 113 comparisons. Generally, if a regular expression pattern has a single alternation construct or a single optional quantifier, the number of comparison operations required to match the pattern is more than twice the number of characters in the input string.

## Backtracking with nested optional quantifiers

The number of comparison operations required to match a regular expression pattern can increase exponentially if the pattern includes a large number of alternation constructs, if it includes nested alternation constructs, or, most commonly, if it includes nested optional quantifiers. For example, the regular expression pattern `^(a+)+$` is designed to match a complete string that contains one or more "a" characters. The example provides two input strings of identical length, but only the first string matches the pattern. The [System.Diagnostics.Stopwatch](xref:System.Diagnostics.Stopwatch) class is used to determine how long the match operation takes. 

```csharp
using System;
using System.Diagnostics;
using System.Text.RegularExpressions;

public class Example
{
   public static void Main()
   {
      string pattern = "^(a+)+$";
      string[] inputs = { "aaaaaa", "aaaaa!" };
      Regex rgx = new Regex(pattern);
      Stopwatch sw;

      foreach (string input in inputs) {
         sw = Stopwatch.StartNew();   
         Match match = rgx.Match(input);
         sw.Stop();
         if (match.Success)
            Console.WriteLine("Matched {0} in {1}", match.Value, sw.Elapsed);
         else
            Console.WriteLine("No match found in {0}", sw.Elapsed);
      }
   }
}
```

```vb
Imports System.Diagnostics
Imports System.Text.RegularExpressions

Module Example
   Public Sub Main()
      Dim pattern As String = "^(a+)+$"
      Dim inputs() As String = { "aaaaaa", "aaaaa!" }
      Dim rgx As New Regex(pattern)
      Dim sw As Stopwatch

      For Each input As String In inputs
         sw = Stopwatch.StartNew()   
         Dim match As Match = rgx.Match(input)
         sw.Stop()
         If match.Success Then
            Console.WriteLine("Matched {0} in {1}", match.Value, sw.Elapsed)
         Else
            Console.WriteLine("No match found in {0}", sw.Elapsed)
         End If      
      Next
   End Sub
End Module
```

As the output from the example shows, the regular expression engine took about twice as long to find that an input string did not match the pattern as it did to identify a matching string. This is because an unsuccessful match always represents a worst-case scenario. The regular expression engine must use the regular expression to follow all possible paths through the data before it can conclude that the match is unsuccessful, and the nested parentheses create many additional paths through the data. The regular expression engine concludes that the second string did not match the pattern by doing the following:

* It checks that it was at the beginning of the string, and then matches the first five characters in the string with the pattern a+. It then determines that there are no additional groups of "a" characters in the string. Finally, it tests for the end of the string. Because one additional character remains in the string, the match fails. This failed match requires 9 comparisons. The regular expression engine also saves state information from its matches of "a" (which we will call match 1), "aa" (match 2), "aaa" (match 3), and "aaaa" (match 4).

* It returns to the previously saved match 4. It determines that there is one additional "a" character to assign to an additional captured group. Finally, it tests for the end of the string. Because one additional character remains in the string, the match fails. This failed match requires 4 comparisons. So far, a total of 13 comparisons have been performed.

* It returns to the previously saved match 3. It determines that there are two additional "a" characters to assign to an additional captured group. However, the end-of-string test fails. It then returns to match3 and tries to match the two additional "a" characters in two additional captured groups. The end-of-string test still fails. These failed matches require 12 comparisons. So far, a total of 25 comparisons have been performed. 

Comparison of the input string with the regular expression continues in this way until the regular expression engine has tried all possible combinations of matches, and then concludes that there is no match. Because of the nested quantifiers, this comparison is an O(2n) or an exponential operation, where n is the number of characters in the input string. This means that in the worst case, an input string of 30 characters requires approximately 1,073,741,824 comparisons, and an input string of 40 characters requires approximately 1,099,511,627,776 comparisons. If you use strings of these or even greater lengths, regular expression methods can take an extremely long time to complete when they process input that does not match the regular expression pattern.

## Controlling backtracking

Backtracking lets you create powerful, flexible regular expressions. However, as the previous section showed, these benefits may be coupled with unacceptably poor performance. To prevent excessive backtracking, you should define a time-out interval when you instantiate a [Regex](xref:System.Text.RegularExpressions.Regex) object or call a static regular expression matching method. This is discussed in the next section. In addition, .NET Core supports three regular expression language elements that limit or suppress backtracking and that support complex regular expressions with little or no performance penalty: [nonbacktracking subexpressions](#nonbacktracking-subexpression), [lookbehind assertions](#lookbehind-assertions), and [lookahead assertions](#lookahead-assertions). For more information about each language element, see [Grouping constructs in regular expressions](grouping.md).

### Defining a time-out interval

You can set a time-out value that represents the longest interval the regular expression engine will search for a single match before it abandons the attempt and throws a [RegexMatchTimeoutException](xref:System.Text.RegularExpressions.RegexMatchTimeoutException) exception. You specify the time-out interval by supplying a [TimeSpan](xref:System.TimeSpan) value to the `Regex(String, RegexOptions, TimeSpan)` constructor for instance regular expressions. In addition, each static pattern matching method has an overload with a [TimeSpan](xref:System.TimeSpan) value to the [Regex.Regex(String, RegexOptions, TimeSpan)] parameter that allows you to specify a time-out value. By default, the time-out interval is set to [Regex.InfiniteMatchTimeout](xref:System.Text.RegularExpressions.Regex.InfiniteMatchTimeout) and the regular expression engine does not time out. 

> [!IMPORTANT]
> We recommend th>at you always set a time-out interval if your regular expression relies on backtracking.
 
A [RegexMatchTimeoutException](xref:System.Text.RegularExpressions.RegexMatchTimeoutException)n exception indicates that the regular expression engine was unable to find a match within in the specified time-out interval but does not indicate why the exception was thrown. The reason might be excessive backtracking, but it is also possible that the time-out interval was set too low given the system load at the time the exception was thrown. When you handle the exception, you can choose to abandon further matches with the input string or increase the time-out interval and retry the matching operation. 

For example, the following code calls the `Regex(String, RegexOptions, TimeSpan)` constructor to instantiate a Regex object with a time-out value of one second. The regular expression pattern `(a+)+$`, which matches one or more sequences of one or more "a" characters at the end of a line, is subject to excessive backtracking. If a [RegexMatchTimeoutException](xref:System.Text.RegularExpressions.RegexMatchTimeoutException) is thrown, the example increases the time-out value up to a maximum interval of three seconds. After that, it abandons the attempt to match the pattern.

```csharp
using System;
using System.ComponentModel;
using System.Diagnostics;
using System.Security;
using System.Text.RegularExpressions;
using System.Threading; 

public class Example
{
   const int MaxTimeoutInSeconds = 3;

   public static void Main()
   {
      string pattern = @"(a+)+$";    // DO NOT REUSE THIS PATTERN.
      Regex rgx = new Regex(pattern, RegexOptions.IgnoreCase, TimeSpan.FromSeconds(1));       
      Stopwatch sw = null;

      string[] inputs= { "aa", "aaaa>", 
                         "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
                         "aaaaaaaaaaaaaaaaaaaaaa>",
                         "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa>" };

      foreach (var inputValue in inputs) {
         Console.WriteLine("Processing {0}", inputValue);
         bool timedOut = false;
         do { 
            try {
               sw = Stopwatch.StartNew();
               // Display the result.
               if (rgx.IsMatch(inputValue)) {
                  sw.Stop();
                  Console.WriteLine(@"Valid: '{0}' ({1:ss\.fffffff} seconds)", 
                                    inputValue, sw.Elapsed); 
               }
               else {
                  sw.Stop();
                  Console.WriteLine(@"'{0}' is not a valid string. ({1:ss\.fffff} seconds)", 
                                    inputValue, sw.Elapsed);
               }
            }
            catch (RegexMatchTimeoutException e) {   
               sw.Stop();
               // Display the elapsed time until the exception.
               Console.WriteLine(@"Timeout with '{0}' after {1:ss\.fffff}", 
                                 inputValue, sw.Elapsed);
               Thread.Sleep(1500);       // Pause for 1.5 seconds.

               // Increase the timeout interval and retry.
               TimeSpan timeout = e.MatchTimeout.Add(TimeSpan.FromSeconds(1));
               if (timeout.TotalSeconds > MaxTimeoutInSeconds) {
                  Console.WriteLine("Maximum timeout interval of {0} seconds exceeded.",
                                    MaxTimeoutInSeconds);
                  timedOut = false;
               }
               else {               
                  Console.WriteLine("Changing the timeout interval to {0}", 
                                    timeout); 
                  rgx = new Regex(pattern, RegexOptions.IgnoreCase, timeout);
                  timedOut = true;
               }
            }
         } while (timedOut);
         Console.WriteLine();
      }   
   }
}
// The example displays output like the following :
//    Processing aa
//    Valid: 'aa' (00.0000779 seconds)
//    
//    Processing aaaa>
//    'aaaa>' is not a valid string. (00.00005 seconds)
//    
//    Processing aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
//    Valid: 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa' (00.0000043 seconds)
//    
//    Processing aaaaaaaaaaaaaaaaaaaaaa>
//    Timeout with 'aaaaaaaaaaaaaaaaaaaaaa>' after 01.00469
//    Changing the timeout interval to 00:00:02
//    Timeout with 'aaaaaaaaaaaaaaaaaaaaaa>' after 02.01202
//    Changing the timeout interval to 00:00:03
//    Timeout with 'aaaaaaaaaaaaaaaaaaaaaa>' after 03.01043
//    Maximum timeout interval of 3 seconds exceeded.
//    
//    Processing aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa>
//    Timeout with 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa>' after 03.01018
//    Maximum timeout interval of 3 seconds exceeded.
```

```vb
Imports System.ComponentModel
Imports System.Diagnostics
Imports System.Security
Imports System.Text.RegularExpressions
Imports System.Threading 

Module Example
   Const MaxTimeoutInSeconds As Integer = 3

   Public Sub Main()
      Dim pattern As String = "(a+)+$"    ' DO NOT REUSE THIS PATTERN.
      Dim rgx As New Regex(pattern, RegexOptions.IgnoreCase, TimeSpan.FromSeconds(1))       
      Dim sw As Stopwatch = Nothing

      Dim inputs() As String = { "aa", "aaaa>", 
                                 "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
                                 "aaaaaaaaaaaaaaaaaaaaaa>",
                                 "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa>" }

      For Each inputValue In inputs
         Console.WriteLine("Processing {0}", inputValue)
         Dim timedOut As Boolean = False
         Do 
            Try
               sw = Stopwatch.StartNew()
               ' Display the result.
               If rgx.IsMatch(inputValue) Then
                  sw.Stop()
                  Console.WriteLine("Valid: '{0}' ({1:ss\.fffffff} seconds)", 
                                    inputValue, sw.Elapsed) 
               Else
                  sw.Stop()
                  Console.WriteLine("'{0}' is not a valid string. ({1:ss\.fffff} seconds)", 
                                    inputValue, sw.Elapsed)
               End If
            Catch e As RegexMatchTimeoutException   
               sw.Stop()
               ' Display the elapsed time until the exception.
               Console.WriteLine("Timeout with '{0}' after {1:ss\.fffff}", 
                                 inputValue, sw.Elapsed)
               Thread.Sleep(1500)       ' Pause for 1.5 seconds.

               ' Increase the timeout interval and retry.
               Dim timeout As TimeSpan = e.MatchTimeout.Add(TimeSpan.FromSeconds(1))
               If timeout.TotalSeconds > MaxTimeoutInSeconds Then
                  Console.WriteLine("Maximum timeout interval of {0} seconds exceeded.",
                                    MaxTimeoutInSeconds)
                  timedOut = False
               Else                
                  Console.WriteLine("Changing the timeout interval to {0}", 
                                    timeout) 
                  rgx = New Regex(pattern, RegexOptions.IgnoreCase, timeout)
                  timedOut = True
               End If
            End Try
         Loop While timedOut
         Console.WriteLine()
      Next   
   End Sub 
End Module
' The example displays output like the following:
'    Processing aa
'    Valid: 'aa' (00.0000779 seconds)
'    
'    Processing aaaa>
'    'aaaa>' is not a valid string. (00.00005 seconds)
'    
'    Processing aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
'    Valid: 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa' (00.0000043 seconds)
'    
'    Processing aaaaaaaaaaaaaaaaaaaaaa>
'    Timeout with 'aaaaaaaaaaaaaaaaaaaaaa>' after 01.00469
'    Changing the timeout interval to 00:00:02
'    Timeout with 'aaaaaaaaaaaaaaaaaaaaaa>' after 02.01202
'    Changing the timeout interval to 00:00:03
'    Timeout with 'aaaaaaaaaaaaaaaaaaaaaa>' after 03.01043
'    Maximum timeout interval of 3 seconds exceeded.
'    
'    Processing aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa>
'    Timeout with 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa>' after 03.01018
'    Maximum timeout interval of 3 seconds exceeded.
```

### Nonbacktracking subexpression

The **(?>** _subexpression_**)** language element suppresses backtracking in a subexpression. It is useful for preventing the performance problems associated with failed matches. 

The following example illustrates how suppressing backtracking improves performance when using nested quantifiers. It measures the time required for the regular expression engine to determine that an input string does not match two regular expressions. The first regular expression uses backtracking to attempt to match a string that contains one or more occurrences of one or more hexadecimal digits, followed by a colon, followed by one or more hexadecimal digits, followed by two colons. The second regular expression is identical to the first, except that it disables backtracking. As the output from the example shows, the performance improvement from disabling backtracking is significant.

```csharp
using System;
using System.Diagnostics;
using System.Text.RegularExpressions;

public class Example
{
   public static void Main()
   {
      string input = "b51:4:1DB:9EE1:5:27d60:f44:D4:cd:E:5:0A5:4a:D24:41Ad:";
      bool matched;
      Stopwatch sw;

      Console.WriteLine("With backtracking:");
      string backPattern = "^(([0-9a-fA-F]{1,4}:)*([0-9a-fA-F]{1,4}))*(::)$";
      sw = Stopwatch.StartNew();
      matched = Regex.IsMatch(input, backPattern);
      sw.Stop();
      Console.WriteLine("Match: {0} in {1}", Regex.IsMatch(input, backPattern), sw.Elapsed);
      Console.WriteLine();

      Console.WriteLine("Without backtracking:");
      string noBackPattern = "^((?>[0-9a-fA-F]{1,4}:)*(?>[0-9a-fA-F]{1,4}))*(::)$";
      sw = Stopwatch.StartNew();
      matched = Regex.IsMatch(input, noBackPattern);
      sw.Stop();
      Console.WriteLine("Match: {0} in {1}", Regex.IsMatch(input, noBackPattern), sw.Elapsed);
   }
}
// The example displays output like the following:
//       With backtracking:
//       Match: False in 00:00:27.4282019
//       
//       Without backtracking:
//       Match: False in 00:00:00.0001391
```

```vb
Imports System.Diagnostics
Imports System.Text.RegularExpressions

Module Example
   Public Sub Main()
      Dim input As String = "b51:4:1DB:9EE1:5:27d60:f44:D4:cd:E:5:0A5:4a:D24:41Ad:"
      Dim matched As Boolean
      Dim sw As Stopwatch

      Console.WriteLine("With backtracking:")
      Dim backPattern As String = "^(([0-9a-fA-F]{1,4}:)*([0-9a-fA-F]{1,4}))*(::)$"
      sw = Stopwatch.StartNew()
      matched = Regex.IsMatch(input, backPattern)
      sw.Stop()
      Console.WriteLine("Match: {0} in {1}", Regex.IsMatch(input, backPattern), sw.Elapsed)
      Console.WriteLine()

      Console.WriteLine("Without backtracking:")
      Dim noBackPattern As String = "^((?>[0-9a-fA-F]{1,4}:)*(?>[0-9a-fA-F]{1,4}))*(::)$"
      sw = Stopwatch.StartNew()
      matched = Regex.IsMatch(input, noBackPattern)
      sw.Stop()
      Console.WriteLine("Match: {0} in {1}", Regex.IsMatch(input, noBackPattern), sw.Elapsed)
   End Sub
End Module
' The example displays the following output:
'       With backtracking:
'       Match: False in 00:00:27.4282019
'       
'       Without backtracking:
'       Match: False in 00:00:00.0001391
```

### Lookbehind assertions

.NET includes two language elements, **(?<**=_subexpression_**)** and **(?<!**_subexpression_**)**, that match the previous character or characters in the input string. Both language elements are zero-width assertions; that is, they determine whether the character or characters that immediately precede the current character can be matched by *subexpression*, without advancing or backtracking. 

**(?<**=_subexpression_**)** is a positive lookbehind assertion; that is, the character or characters before the current position must match *subexpression*. **(?<!**_subexpression_**)** is a negative lookbehind assertion; that is, the character or characters before the current position must not match *subexpression*. Both positive and negative lookbehind assertions are most useful when *subexpression* is a subset of the previous *subexpression*. 

The following example uses two equivalent regular expression patterns that validate the user name in an e-mail address. The first pattern is subject to poor performance because of excessive backtracking. The second pattern modifies the first regular expression by replacing a nested quantifier with a positive lookbehind assertion. The output from the example displays the execution time of the [Regex.IsMatch](xref:System.Text.RegularExpressions.Regex.IsMatch(System.String,System.String,System.Text.RegularExpressions.RegexOptions)) method.

```csharp
using System;
using System.Diagnostics;
using System.Text.RegularExpressions;

public class Example
{
   public static void Main()
   {
      Stopwatch sw;
      string input = "aaaaaaaaaaaaaaaaaaaa";
      bool result;

      string pattern = @"^[0-9A-Z]([-.\w]*[0-9A-Z])?@";
      sw = Stopwatch.StartNew();
      result = Regex.IsMatch(input, pattern, RegexOptions.IgnoreCase);
      sw.Stop();
      Console.WriteLine("Match: {0} in {1}", result, sw.Elapsed);

      string behindPattern = @"^[0-9A-Z][-.\w]*(?<=[0-9A-Z])@";
      sw = Stopwatch.StartNew();
      result = Regex.IsMatch(input, behindPattern, RegexOptions.IgnoreCase);
      sw.Stop();
      Console.WriteLine("Match with Lookbehind: {0} in {1}", result, sw.Elapsed);
   }
}
// The example displays output similar to the following:
//       Match: True in 00:00:00.0017549
//       Match with Lookbehind: True in 00:00:00.0000659
```

```vb
Module Example
   Public Sub Main()
      Dim sw As Stopwatch
      Dim input As String = "aaaaaaaaaaaaaaaaaaaa"
      Dim result As Boolean

      Dim pattern As String = "^[0-9A-Z]([-.\w]*[0-9A-Z])?@"
      sw = Stopwatch.StartNew()
      result = Regex.IsMatch(input, pattern, RegexOptions.IgnoreCase)
      sw.Stop()
      Console.WriteLine("Match: {0} in {1}", result, sw.Elapsed)

      Dim behindPattern As String = "^[0-9A-Z][-.\w]*(?<=[0-9A-Z])@"
      sw = Stopwatch.StartNew()
      result = Regex.IsMatch(input, behindPattern, RegexOptions.IgnoreCase)
      sw.Stop()
      Console.WriteLine("Match with Lookbehind: {0} in {1}", result, sw.Elapsed)
   End Sub
End Module
' The example displays output similar to the following:
'       Match: True in 00:00:00.0017549
'       Match with Lookbehind: True in 00:00:00.0000659
```

The first regular expression pattern, `^[0-9A-Z]([-.\w]*[0-9A-Z])*@, is defined as shown in the following table.`

Pattern | Description
------- | ----------- 
`^` | Start the match at the beginning of the string.
`[0-9A-Z]` | Match an alphanumeric character. This comparison is case-insensitive, because the [Regex.IsMatch](xref:System.Text.RegularExpressions.Regex.IsMatch(System.String,System.String,System.Text.RegularExpressions.RegexOptions)) method is called with the [RegexOptions.IgnoreCase](xref:System.Text.RegularExpressions.RegexOptions.IgnoreCase) option.
`[-.\w]*` | Match zero, one, or more occurrences of a hyphen, period, or word character. 
`[0-9A-Z]` | Match an alphanumeric character. 
`([-.\w]*[0-9A-Z])*` | Match zero or more occurrences of the combination of zero or more hyphens, periods, or word characters, followed by an alphanumeric character. This is the first capturing group.
`@` | Match an at sign ("@").
 
The second regular expression pattern, `^[0-9A-Z][-.\w]*(?<=[0-9A-Z])@`, uses a positive lookbehind assertion. It is defined as shown in the following table.

Pattern | Description
------- | ----------- 
`^` | Start the match at the beginning of the string.
`[0-9A-Z]` | Match an alphanumeric character. This comparison is case-insensitive, because the [Regex.IsMatch](xref:System.Text.RegularExpressions.Regex.IsMatch(System.String,System.String,System.Text.RegularExpressions.RegexOptions)) method is called with the [RegexOptions.IgnoreCase](xref:System.Text.RegularExpressions.RegexOptions.IgnoreCase) option.
`[-.\w]*` | Match zero or more occurrences of a hyphen, period, or word character.
`(?<=[0-9A-Z])` | Look back at the last matched character and continue the match if it is alphanumeric. Note that alphanumeric characters are a subset of the set that consists of periods, hyphens, and all word characters.
`@` | Match an at sign ("@").
 
### Lookahead assertions

.NET includes two language elements, **(?**=_subexpression_**)** and **(?!**_subexpression_**)**, that match the next character or characters in the input string. Both language elements are zero-width assertions; that is, they determine whether the character or characters that immediately follow the current character can be matched by *subexpression*, without advancing or backtracking. 

**(?**=_subexpression_**)** is a positive lookahead assertion; that is, the character or characters after the current position must match *subexpression*. **(?!**_subexpression_**)** is a negative lookahead assertion; that is, the character or characters after the current position must not match *subexpression*. Both positive and negative lookahead assertions are most useful when *subexpression* is a subset of the next *subexpression*. 

The following example uses two equivalent regular expression patterns that validate a fully qualified type name. The first pattern is subject to poor performance because of excessive backtracking. The second modifies the first regular expression by replacing a nested quantifier with a positive lookahead assertion. The output from the example displays the execution time of the [Regex.IsMatch](xref:System.Text.RegularExpressions.Regex.IsMatch(System.String,System.String,System.Text.RegularExpressions.RegexOptions)) method.

```csharp
using System;
using System.Diagnostics;
using System.Text.RegularExpressions;

public class Example
{
   public static void Main()
   {
      string input = "aaaaaaaaaaaaaaaaaaaaaa.";
      bool result;
      Stopwatch sw;

      string pattern = @"^(([A-Z]\w*)+\.)*[A-Z]\w*$";
      sw = Stopwatch.StartNew();
      result = Regex.IsMatch(input, pattern, RegexOptions.IgnoreCase);
      sw.Stop();
      Console.WriteLine("{0} in {1}", result, sw.Elapsed);

      string aheadPattern = @"^((?=[A-Z])\w+\.)*[A-Z]\w*$";
      sw = Stopwatch.StartNew();
      result = Regex.IsMatch(input, aheadPattern, RegexOptions.IgnoreCase);
      sw.Stop();
      Console.WriteLine("{0} in {1}", result, sw.Elapsed);
   }
}
// The example displays the following output:
//       False in 00:00:03.8003793
//       False in 00:00:00.0000866
```

```vb
Imports System.Diagnostics
Imports System.Text.RegularExpressions

Module Example
   Public Sub Main()
      Dim input As String = "aaaaaaaaaaaaaaaaaaaaaa."
      Dim result As Boolean
      Dim sw As Stopwatch

      Dim pattern As String = "^(([A-Z]\w*)+\.)*[A-Z]\w*$"
      sw = Stopwatch.StartNew()
      result = Regex.IsMatch(input, pattern, RegexOptions.IgnoreCase)
      sw.Stop()
      Console.WriteLine("{0} in {1}", result, sw.Elapsed)

      Dim aheadPattern As String = "^((?=[A-Z])\w+\.)*[A-Z]\w*$"
      sw = Stopwatch.StartNew()
      result = Regex.IsMatch(input, aheadPattern, RegexOptions.IgnoreCase)
      sw.Stop()
      Console.WriteLine("{0} in {1}", result, sw.Elapsed)
   End Sub
End Module
' The example displays the following output:
'       False in 00:00:03.8003793
'       False in 00:00:00.0000866
```

The first regular expression pattern, `^(([A-Z]\w*)+\.)*[A-Z]\w*$`, is defined as shown in the following table.

Pattern | Description
------- | ----------- 
`^` | Start the match at the beginning of the string.
`([A-Z]\w*)+\.` | Match an alphabetical character (A-Z) followed by zero or more word characters one or more times, followed by a period. This comparison is case-insensitive, because the [Regex.IsMatch](xref:System.Text.RegularExpressions.Regex.IsMatch(System.String,System.String,System.Text.RegularExpressions.RegexOptions)) method is called with the [RegexOptions.IgnoreCase](xref:System.Text.RegularExpressions.RegexOptions.IgnoreCase) option.
`(([A-Z]\w*)+\.)*` | Match the previous pattern zero or more times. 
`[A-Z]\w*` | Match an alphabetical character followed by zero or more word characters.
`$` | End the match at the end of the input string.
 

The second regular expression pattern, `^((?=[A-Z])\w+\.)*[A-Z]\w*$`, uses a positive lookahead assertion. It is defined as shown in the following table.

Pattern | Description
------- | ----------- 
`^` | Start the match at the beginning of the string.
`(?=[A-Z])` | Look ahead to the first character and continue the match if it is alphabetical (A-Z). This comparison is case-insensitive, because the [Regex.IsMatch](xref:System.Text.RegularExpressions.Regex.IsMatch(System.String,System.String,System.Text.RegularExpressions.RegexOptions)) method is called with the [RegexOptions.IgnoreCase](xref:System.Text.RegularExpressions.RegexOptions.IgnoreCase) option.
`\w+\.` | Match one or more word characters followed by a period.
`((?=[A-Z])\w+\.)*` | Match the pattern of one or more word characters followed by a period zero or more times. The initial word character must be alphabetical. 
`[A-Z]\w*` | Match an alphabetical character followed by zero or more word characters.
`$` | End the match at the end of the input string.
 
## See also

[.NET regular expressions](regular-expressions.md)

[Regular expression language - quick reference](quick-ref.md)

[Quantifiers in regular expressions](quantifiers.md)

[Alternation constructs in regular expressions](alternation.md)

[Grouping constructs in regular expressions](grouping.md)

