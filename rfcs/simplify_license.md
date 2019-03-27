## RFC #18: Simplify License

### Summary

Reduce WPT's license to the 3-Clause BSD License and update the copyright
holder to "web-platform-tests contributors."

### Details

WPT is currently dual-licensed under [the W3C Test Suite
License](#w3c-test-suite-license) and [the W3C 3-clause BSD
License](#w3c-3-clause-bsd-license). Both licenses name "W3C" as the copyright
holder.

In [WPT issue gh-11009](https://github.com/web-platform-tests/wpt/issues/11009)
Philippe Le Hegaret (W3C) led a discussion about how this dual license is no
longer appropriate for the web-platform-tests project. Wendy Seltzer (W3C)
proposed changes to the Patents and Standards Interest Group (PSIG) and
reported:

> Per conversation with [Philip Jägenstedt (Google)] I'm filing two pull
> requests against license.md, one dropping from the dual license to the W3C
> 3-clause BSD; the other replacing with the straight 3-clause BSD (which
> differs only in not having the named copyright holder W3C). PSIG has given
> non-objection to the W3C 3-clause BSD. If that would still cause legal
> hiccups, then we can probably persuade them of the vanilla 3-clause.

Because [Philippe Le Hegaret and Philip Jägenstedt have made it known that
naming "W3C" as the copyright holder is
undesirable](https://github.com/web-platform-tests/wpt/pull/11191), we choose
to adopt [the 3-Clause BSD
License](https://opensource.org/licenses/BSD-3-Clause) which lists the
copyright holder as "web-platform-tests contributors." The complete license
text is as follows:

> # The 3-Clause BSD License
>
> Copyright 2019 web-platform-tests contributors
>
> Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:
>
> 1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
> 2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
> 3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.
>
> THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

### Risks

Pending the endorsement from the W3C Patents and Standards Interest Group,
there appears to be no risk associated with this change.
