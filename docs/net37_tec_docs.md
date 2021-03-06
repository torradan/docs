```ASCIIDOC











              Network Technical Documentation


                       version 2.0.37








                             by



                       Wayne Heyward
                  aka Midnight Tree Bandit
                   Updated for Net37, WSS








   Copyright (c) 1998-2000, WWIV Software Services, LLC
               Copyright (c) 1995, Wayne Bell

 




                     Table of Contents



    I.  INTRODUCTION.................................  1

   II.  GENERAL OVERVIEW OF WWIV NETWORKING..........  4

  III.  MAKING CONNECTIONS...........................  6

   IV.  MESSAGE PACKETS.............................. 12

    V.  BBSLIST/CONNECT FILES AND MESSAGE ROUTING.... 28

   VI.  TIPS FOR WRITING WWIVNET SOFTWARE............ 37

        APPENDIX A................................... 41






























                            -i-
 


I.   INTRODUCTION

 A.   History

  Back in 1989, Wayne Bell released the first technical
  documentation covering the technical workings of the
  WWIV networking software.  While much of the information
  in that document is still relevant now, much has changed
  since 1989.  The Group structure has been added, support
  for more message types, and support for preprocessors to
  the packet tossers has been added.

  So in late 1992 or early 1993 Wayne asked for volunteers
  to rewrite the Network Technical documentation.  No one
  spoke up. Then in March I started providing WWIV support
  on the GEnie information service, and some people started
  asking about getting technical information so they could
  get their non-WWIV boards to communicate with WWIV
  networks.  Looking around, I found the original doc, and
  asked Wayne if anyone had answered his call.  As it
  turned out, I became the volunteer.  After much
  procrastination and "How's the doc coming?" from Wayne,
  here it is.

 B.   Purpose, Limitations, and Other Info

  The purpose for this document is to explain WWIV based
  networking interface for those who wish to write software
  which communicates with WWIV based systems, either
  independently or as an extra utility for the existing
  NETXX software.

  The documentation that you are reading now is more an
  expansion and clarification of the original docs than it
  is a total rewrite.  It looks a little neater too, thanks
  to Word Perfect. Now that the bulk of the work is done,
  the plan is to update them with each release of NETxx to
  reflect new features.  The information in this document
  is current as of the Net37 release (NET37).

  First clarification: in the context of this document, the
  term "NETXX" refers to the software for interfacing
  WWIV-type networks and any network which uses that
  software.

  This documentation assumes that the reader at least: a)

                            -1-
 


  has a good working knowledge of C and the various C data
  types and data structures. b)   has a general familiarity
  of how file transfer protocols work.

  Due to the proprietary nature of the NETXX software, we
  will not cover the inner workings of the current networking
  programs distributed in NET??.ZIP.  We will only discuss
  the external requirements for any third  party interfaces
  that may be written for connecting non-WWIV systems to
  WWIV networks. Likewise, we will not discuss the NETUP
  software, which generates and sends out the network node
  lists and connect lists.  You'll just have to figure that
  out for yourself.

  This document is not a replacement for the NetXX
  Software Documentation.  This doc does not describe how
  to use the network software, only how it works.  For
  instructions on using the NETXX software and familiarize
  yourself with how to use it, see the documentation,
  which can be found in the current NetXX release.

  A note on the numbering for this document.  This is
  version 2.0.37.  The first number indicates that this is
  a major rewrite from the original by Wayne Bell.  Unless
  a major overhaul is done, this is not likely to change.
  The 0 in the second part indicates that this is the first
  version of this document.  If there are any major changes
  or additions made, this will be incremented.  The 37
  indicates that this information is current as of version
  37 of the NETXX software (NET37).  Minor changes
  reflecting small interface changes in the NETXX
  software will cause this number to be changed.  This
  number will not always be the same as the latest version
  of the NETXX software; if there are no changes in the
  external interface, this document will not be updated.

  Comments concerning this document are welcome as are
  suggestions for additional information to include,
  things that could be explained more fully, and so
  forth.  Send comments and suggestions to wss@wwiv.com
  or 1@50 on most major WWIV based networks.



                            -2-
 

 C.   Relevant Copyrights and Acknowledgements
  This document is Copyright 1994 by Wayne Heyward (aka
  Midnight Tree Bandit) and updated by WWIV Software
  Services (WSS).  It may be freely distributed provided
  it is not altered in any way.  This is copyrighted to
  prevent unauthorized (and possibly inaccurate) changes
  from being made to this document by anyone other than
  myself or any other appointed by WWIV Software Services (
  should I be unable to continue updating this document).

  WWIV BBS and the NETXX software (distributed as NETxx)
  are Copyright 1989-1995 by Wayne Bell
  and Copyright 1998-2000, WWIV Software Services, LLC

  NETEDIT is Copyright 1994 by Black Dragon Enterprises.
          and Copyright 1999-2000, BRware Software

  DSZ is Copyright 1994 by Omen Technology INC.

  HSLink is Copyright 1994 by Samuel H. Smith.

  The PKWare Compression Libraries are copyright 1993 by
  PKWARE, Inc.

  The NETXX interface information and code in this
  document is placed in the public domain, and may be
  freely used for the purpose of interfacing with WWIV
  networks and network software.

  I would like to thank Wayne Bell, not only for creating a
  top- notch BBS program that is both powerful and easy to
  use, but creating a networking scheme that is more
  painless to set up and operate than any other out there.
  He has said often that if he knew then what he does now,
  things would have been different.  I cannot help thinking
  that the result would have been less elegant or easy to
  use.

  I would also like to thank Wayne for his patience over
  the last year and a half, waiting for me to get this
  document started. I've fired thousands of questions at
  him the last few weeks in an effort to make this
  documentation as complete as possible, and he answered
  every one.

  And finally thanks to Filo, who also provided vital
  information and advice without which this documentation
  would be incomplete.





                            -3-
 


II.  GENERAL OVERVIEW OF WWIV NETWORKING

 A WWIV network is basically a loose confederation of WWIV
 BBS systems that use the NETXX (or compatible) software.

 The software does not limit the connection structure, so
 the member sysops can connect to anyone they wish (subject
 to the rules of their network).  For ease of
 administration, the network may be split up into Groups,
 each with their own coordinator.  Node numbers are an
 unsigned short int, so that nodes may be assigned a value
 from 1 to 65535.

 The lists of nodes are distributed in two sets of files:
 BBSLIST.xxx and CONNECT.xxx.  There are two different ways
 of handling these files.  The old way has just one
 BBSLIST.NET and one CONNECT.NET file. The CONNECT.NET file
 assigns costs to each connection for each system. Some
 smaller networks still use this setup.  The new way,
 implemented in 1990, uses several BBSLIST and CONNECT
 files with extensions indicating group number (e.g. Group
 1's files would be BBSLIST.1 and CONNECT.1).  BBSLIST.0
 contains a list of Group numbers, and CONNECT.0 contains
 inter-Group connections.  It is important that any
 non-WWIV systems be able to support both setups if they
 wish to connect to a WWIV network.

 Like all BBS networks, the primary purpose is to exchange
 private mail and public posts between BBSes.  The passing
 of files in binary form, however, is not currently
 supported by the NETXX software. There are some third party
 programs and in fact, an entire network which can handle
 the network transfer of files.  All messages also have a
 maximum size limit of 32k.

 The basic NETXX software distributed in NETxx.ZIP by WSS
 consists of five programs: NETWORK.EXE, NETWORKF.EXE,
 NETWORK1.EXE, NETWORK2.EXE and NETWORK3.EXE.  Any
 alternative software  for interfacing with WWIV
 should follow the same structure.

 NETWORK.EXE handles connections between systems.  On the
 sending end of the connection, NETWORK.EXE calls out to
 another system, chosen from the CALLOUT.NET file on that
 system.  The answering system activates NETWORK.EXE when
 it detects a network call.  Once they're talking, they

                            -4-
 


 make any mail packet transfers needed.

 NETWORKF.EXE is a FOSSIL version of NETWORK.EXE.  If
 your system operates on V4.30 or later of the BBS software
 or you have an installed FOSSIL driver, you should delete
 NETWORK.EXE and rename this file to NETWORK.EXE. You must
 also have the freeware FOSSIL protocol CEXYZ properly
 installed for this version of NETWORK.EXE to operate.
 If you are running an earlier version of the BBS or do
 not use a FOSSIL driver, this file may be deleted.

 NETWORK1.EXE is the first of the two mail tossers.  This
 one takes the incoming packet received by NETWORK.EXE and
 distributed the messages within to their rightful places.
 Local mail goes into LOCAL.NET, while mail passing through
 to other systems is tossed into packet files for the next
 hop.

 NETWORK2.EXE tosses the local messages in LOCAL.NET.  Most
 are messages for email or local subboards, but there are
 also network updates, sub REQuests, software "pings," and
 other special purpose message types.  It also has the
 ability to call on third-party preprocessors for special
 handling of certain types of messages.

 NETWORK3.EXE processes all the network updates that come
 in.  This helps determine what routing off-system mail
 will take.

 Each WWIV network also has its own special encoding and
 decoding programs for handling of updates and network mail
 from the Network Coordinator and Group Coordinators.
 These are the DEmmm.EXE files (mmm corresponding to the
 message type).  The DEmmm.EXE cannot be replaced, so any
 NETWORK2.EXE replacement must be able to recognize the
 need for calling the appropriate DEmmm.EXE, as described
 below.











                            -5-
 


III. MAKING CONNECTIONS

 A.   Establishing Contact

  Through some process, one system decides to call another.
  Various techniques can be used for deciding when and who
  to call; that should have no effect on the network.  Any
  NETXX system should be able to receive a network
  connection at any time (i.e., there is no network mail
  hour).  For an example of a method for limiting call out
  times, see the NETXX Software Docs.

  However the decision is made, a call is made to another
  NETXX system.  If he connects, the connection protocol
  is then begun.

  After connection, the caller waits for the string "NN:".
  To get to this prompt, the caller sends the string "N "
  ('N', space), every half second.  Every 7 seconds, the
  caller sends a carriage return (char 13 decimal).  The
  reason for sending "N " is twofold:  1) the N is to
  answer any yes/no question that may occur on connection;
  and 2) the space is to abort any message printed out to
  callers, if the BBS allows it.  After 16.5 seconds, the
  caller gives up and assumes he is at the NN: prompt.

  Since this sequence is designed specifically with WWIV
  systems in mind, any BBS or front end program receiving a
  call from a NETXX system must be at its login prompt,
  or have already started the network response program
  (such as NETWORK.EXE), by 16.5 seconds after the modem
  connection is estabilshed.  Whether or not it can respond
  to the "N " sequence does not matter as long as it is
  prepared for the next step.  Likewise, any network
  program calling a NETXX system must follow the above
  sequence to ensure a proper connection.

  After the NN: prompt is reached, the caller sends the
  string "\030!-@NETWORK@-!\r" (Ctrl-x, "!-@NETWORK@-!",
  return), to identify that this is the network calling.
  The caller then waits for the net identification string.
  The "!-@NETWORK@-!" string is sent every 6 seconds.
  After 20 seconds without receiving the network
  identification string, the caller gives up and hangs up.

  The network identification string is sent by the
  answering system after getting the "!-@NETWORK@-!"
  string.  The network ident string is sent every 6

                            -6-
 


  seconds, until the network response is received.  The
  answering system gives up and hangs up after 20 seconds.

  The network ident string is:

       \x1b[8mxX33211\x0c31412718\x1b[8DNETWORK!
 (i.e.,
 ESC "[8mxX33211" Ctrl-L "31412718" ESC "[8DNETWORK!")

  The network response string from the calling system is:

       \x1b[8mxyzzy$.@83\x0c##
  (i.e. ESC "[8mxyxxy$.@83" Ctrl-L "##")

  After getting the network identification string, the
  caller sends the network response string every 6 seconds.
  If, after 20 seconds, the answering system does not
  receive a Ctrl-B ('\002\'), the calling system gives up
  the attempt and hangs up. If the Ctrl-B is received, the
  caller considers the connection established.

  After getting the network response string, the answering
  system sends three Ctrl-B characters ("\002\002\002"),
  and considers the two systems connected.
          
 B.   Data Exchange

  After the two systems are connected, the caller sends an
  identification block to the answering system.  The block
  is 128 bytes long, and is sent in the same method as an
  xmodem block of data is sent, complete with CRC.  The
  caller does not wait for a NAK before beginning
  transmission, but begins transmission of the block
  immediately after receiving one Ctrl-B.  The block is
  sent as block #0, with CRC and 128 bytes of data.  After
  receiving the block, the answering system sends the usual
  ACK/NAK, etc.  If after 6 tries the block is not received
  correctly, both sides give up and disconnect.

  The block of data contains from seven to nine pieces of
  data, each separated by a space character (char 32
  decimal).  All bytes after the last piece of data have a
  value of zero.  The pieces of data are:

  system number  Net node number of the sender.

  sendback


                            -7-
 


    Whether the answering system may send a net file back
    to the sender ("0" for no, or "1" for yes).  The caller
    will ignore this field when the answering system sends
    its ident block.  In other words, the only system that
    determines whether sendback is allowed is the system
    making the call.

  cost

   The cost for the call, in cents per minute, as defined
   in CONNECT.NET under the old NETXX setup.  If
   the network is using the Group setup, this
   value will be zero ("0").

  features

   A long integer, sent as a literal, specifying the
   features supported by that system.  Right now, five
   values are possible, affecting mainly the protocol used
   in transfer.  It is read as a bitmap ('fea
   tures&[value]'), with the bits having the following
   effects:
   1 -- Zmodem available (DSZ or Visual Zmodem);
   2 -- numk and netver found after password (see below);
   4 -- HSLink (bidirectional protocol) available.
   Current versions of the NETXX software start with the
   2 bit set, then sets the 1 and 4 bits as  appropriate.
   If neither Zmodem nor HSLink are available, transfer
   defaults to internal Ymodem.

  tosendlen

   The number of bytes in the mail file that is to be
   transferred, sent as literal.

  password

   The password to be used between the two systems.  If no
   password has been set yet, this is set to "NONE". Usual
   passwords are 15 characters long.

  numk

   Number of KB the sender is able to take, sent as a
   literal.  This is calculated as free space on the data
   drive divided by three.  This field will only be
   present if features&2 is set.


                            -8-
 


  net_ver

   Version of the network software, such as "34" for
   NET34.  Third-party calling programs must send the
   version of NETXX software it is compatible with.
   This field will only be present if features&2 is set.

  net_name

   Name of the network the call is for.  This is used  by
   the receiver to determine which data directory the
   network files go into.  This is not a required field,
   but to be safe it should always be included.  One case
   in which it would be necessary is if a BBS is connect
   ing two nodes with the same node number on different
   WWIV networks (i.e., one is @3050.IceNET and the other
   is @3050.TerraNet).
 
  See below for examples of identification blocks sent
  between systems.  To assure that the identification block
  is compatible with that generated by NETWORK.EXE in
  NET37, here is how it's generated and sent out by
  NETWORK.EXE:

    for (i=0; i<128; i++)
      b[i]=0;
     /* 'b' is the net ident string */
    itoa(net_sysnum,b,10);
     /* net_sysnum = system's node number */
    strcat(b," ");
    itoa(sendback,s,10);
    strcat(b,s);
    strcat(b," ");
    itoa((int) (cost*100.0),s,10);
    strcat(b,s);
    strcat(b," ");
    ltoa(features,s,10);
    strcat(b,s);
    strcat(b," ");
    ltoa(tosendlen,s,10);
    strcat(b,s);
    strcat(b," ");
    if (password[0])
      strcat(b,password);
    else
      strcat(b,"NONE");
    if (features & 2) {
      /* 'net_data' is network data dir, FREE_FACT = 3 */

                            -9-
 


      ltoa((long) (freek1(net_data) / FREE_FACT), s, 10);
      strcat(b," ");
      strcat(b,s);
      strcat(b," ");
      itoa(NET_VER,s,10);
      strcat(b,s);
    }
    strcat(b," ");
    strcat(b,net_name);

  After the caller sends the block of data to the answering
  system, the answering system checks the password with the
  password expected from that system.  If the password is
  wrong, or calls from that system are not allowed, the
  answering system discon nects.  Information on passwords
  and allowable systems should be in a separate data file,
  such as the CALLOUT.NET

  If the password is OK, the answering system then sends a
  similar block of data back to the caller, giving its
  system number, features, and bytes to send.  Also, if no
  password had been set yet (the password received from the
  caller is "NONE"), then the answering system randomly
  picks a password and puts it in the password field.

  After both systems have sent and received the data
  blocks, transfers begin.  If the caller has any data to
  send, it is sent first.  If the answering has any data to
  send, it is sent second if the sendback flag is set on.
  It should be noted at this point that since Ymodem is the
  default protocol in the absence of Zmodem or HSLink, it
  must be supported internally in the NETXX calling
  program, or at least be able to run it externally.

  The Ymodem used by the NETXX software  is not
  quite the true Ymodem defined by Forsberg.  There is a
  block #0 which contains the filename, timestamp, and size
  (in bytes), followed by the file itself, ending with an
  EOT block.  There is no "end of batch" block, however.

  If the caller has no data to send, the answering system
  immedi ately begins sending data (again, if sendback is
  set on).  If neither has any data to send, the transfer
  protocol is not started.  Once a netmail file is
  completely sent, it is erased on the sending system. If
  the file is not completely sent (due to excessive errors
  or carrier loss), the file is not deleted, so that
  another attempt can be made on a later call.

                            -10-
 


  Of course, if HSLink is enabled (both ends can use it),
  both netmail files are sent at the same time, if there
  are two.  If only one of the connected systems has a
  netmail file, HSLink is still used.

  After transfers (if any) are done, both systems wait
  0.5-1.0 second, then hang up.  The transfer of files is
  then done.
          
 C.   Identification Block Examples

  1.   System 1 calls system 2 on NetworkX.  System 1 has
  100KB to send to system 2, supports Zmodem and can accept
  400KB. System 2 has 200KB to send to system 1, does not
  support Zmodem, and can accept 1200KB.  The call costs
  $0.10/min (old CONNECT.NET being used).  No password has
  been set yet on system 1, sendback is allowed, and NET37
  is in use on both ends.

  System #1 sends:

       1 1 10 3 102400 NONE 409600 37 NetworkX

  System #2 sends:

       2 1 10 2 204800 ARANDOMPASSWORD 1228800 37 NetworkX

  Since Zmodem is not supported by both systems, Xmodem-1k
  is used to transfer the mail.

  2.   System 10 calls system 20 in IceNET.  System 10 has
  56KB to send, can accept 3485KB, and uses NET36.  System
  20 has 678KB to send, can accept 10MB, and is a PCBoard
  system using software compatible with NET37.  Both
  support Zmodem and the selected password is
  "PASSWORDPASSWOR".  Sendback is not enabled.

  System #10 sends:

       10 0 0 3 57344 PASSWORDPASSWOR 3568640 36 IceNET

  System #20 sends:

       20 0 0 3 0 PASSWORDPASSWOR 10240000 37 IceNET

  Since sendback is not allowed (the caller decides this),
  system 20 won't send any data to system 10, hence the
  number of bytes to send is zero.  Zmodem will be used

                            -11-
 


  since both ends support it.

  3.   System 100 calls system 200 on MTBnet.  System 10
  has 125KB to send, and can accept 300KB.  System 200 has
  350KB to send and can accept 500KB.  Both have HSLink
  available, use NET37, and  use the password
  "OURPASSWORD1234".  Sendback is enabled.

  System #100 sends:

       100 1 0 6 128000 OURPASSWORD1234 307200 37 MTBnet

  System #200 sends:

       200 1 0 6 358400 OURPASSWORD1234 512000 37 MTBnet

  Since system 100 cannot accept the 350KB system 200 has
  for it, only the 125KB from system 1 is sent.  HSLink is
  used for the transfer since both systems support it,
  though only one system is sending a file.
               
IV.  MESSAGE PACKETS

 The most important component of any network is the mail
 file, which contains all the public posts, email, and
 network information which makes the BBS network what it
 is.  NETXX mail files consist of a series of mail
 packets, each with its own header segment describing the
 type and size of the packet.

 There are two types of mail files in NETXX, each similar
 but processed differently.  The netmail file is the file
 received from another system, and contains packets
 destined for that system as well as other systems in the
 network (if the BBS has more than one connect).  Netmail
 files may be compressed, using the PKWare compres sion
 libraries.  The local mail file contains the packets from
 the netmail file which are destined for the BBS only.

 A.   Message Header

  Each message sent through the network has a header.  The
  header tells which user/system originated the message,
  where it is to be sent to, the type of message, and other
  information.  The structure of the header is:

  typedef struct {
       unsigned short tosys,

                            -12-
 

                      touser,
                      fromsys,
                      fromuser;
       unsigned short main_type,
                      minor_type;
       unsigned short list_len;
       unsigned long  daten;
       unsigned long  length;
       unsigned short method;
  } net_header_rec;

  Each header is 24 bytes long.  The fields, in detail,
  are:

  tosys, touser

   The destination of this message (system number and user
   number, if applicable).  The touser field will be zero in
   all cases except for email (main_type==2) and SSMs
   (main_type==15).

  fromsys, fromuser

   The origin of the message (system number and user number).
   This contains the user number/system number combination of
   who actually wrote the message.

  main_type, minor_type

   The type of message this is.  The main type stores the
   actual type (email, post), and the minor_type is used to
   specify what sub-type it is.  For example, the main_type for
   a post is 3.  The minor_type is then used to specify what
   type of post it is, what subboard the post is to go on to.
   A list of main and minor types is found in section IV(D),
   below.

  daten

   The time the message was sent, stored in unix time, as the
   number of seconds since Jan 1, 1970.

  length

   The length of the message.  This includes any information
   between the packet header and the message itself, such as
   the sender's name, message title, and so forth.  Note that
   the length does not include the node list indicated by
   list_len.

                            -13-
 


  method

   If the file is encrypted, compressed, or source-verified,
   this describes the type of compression or encryption used.
   This will tell NETWORK2 (or other local mail tosser) which
   DEmmm.EXE to execute.  DEmmm.EXE is explained in more detail
   in the next section, below.

  list_len

   Some messages need to go to more than one system.  For
   example, networked posts may go to over 20 different
   systems.  It makes no sense to have a separate copy of
   the message for each destination system, so the same
   copy of the header and message is used.  (This is
   referred to as "stacking" the message).  The list_len
   specifies the number of destination systems listed.  If
   list_len is non-zero, then the touser and tosys fields
   are ignored.  The list_len is not used for e-mail to a
   user (main_type is 2 or 7).

   When a message has only one destination system, the
   destina tion system is stored in tosys, and list_len is
   zero.  If there are two or more destinations, then tosys
   is 0, and list_len holds the number of destination
   systems.

   When list_len is non-zero, the list of destination
   systems is stored immediately after the header, but
   before the actual message itself.  The length of the
   list is not included in the length field in the header;
   the length stored in the header is the length of the
   message only.

   Each entry in the destination system list is two bytes
   long, and is stored in the standard byte-reversed format
   of 80x86 chips.

   For example, if a post is destined to system 1, the
   tosys field will be 1, and list_len will be 0.  If the
   post is destined to systems 1 and 2, tosys will be 0,
   list_len will be 2, and there will be 4 bytes between
   the header and message.  The bytes will be: 01 00 02 00
   (remember, byte-reversed format).  The rest of the
   header will be the same for both messages.

  A packet thus consists of a net header, a destination
  list (if any), and the text of the message.  The length

                            -14-
 


  of a full message packet is thus: 24 + 2*list_len +
  msg_length.

  The message text (the part following the header) for a
  post or email begins with information intended for the
  message header shown when the message is displayed.  Each
  piece of information is followed by a carriage return and
  line feed (cr/lf) character to separate it from the next
  except for the message title, which is followed by a NUL
  character.  For most posts and email, that information
  is:

  Message Title

    Whatever title the user gave to the post.

  Sender name

    usually the name and number of the user who wrote
    the message with the system number, in whatever
    format the sending BBS uses.

  Date string

   Formatted date the post was written, in whatever
   format the BBS uses.

  So the message header format for most posts and email
  would be
  TITLE<nul>SENDER_NAME<cr/lf>DATE_STRING<cr/lf>MESSAGE_TEXT.
  Some main_types have other information, as noted in the
  main_type descriptions in section IV(D).

 B.  Mail File Processing

  A NETXX file is simply several packets appended into
  one file. Starting with NET25, the NETXX software
  supports compression of the netmail files to help save on
  connection time in long distance connections, using the
  PKWare Compression Libraries. These files have a slightly
  different format from uncompressed files, but the most
  important issue at this point is that the first long int
  of a compressed file has the value 0xFFFEFFFE.  If you
  purchase the compression libraries from PKWare, details
  covering compressed packets are found in Appendix A.
  Otherwise, anyone using NETXX compatible software
  should be advised to make sure their connect only sends
  them uncompressed files, and the software should be able

                            -15-
 


  to detect and reject compressed packets before attempting
  to process them.  Since there is nothing in the data
  exchange described above to warn that an incoming packet
  is compressed, there is no way to detect and prevent
  transfer of a compressed mail file.

  Once it has been received and (if necessary)
  uncompressed, the mail file should be processed following
  these steps:

  1.   Open the file, and set the current message pointer
       to the beginning of the file.

  2.   Read in the net header (24 bytes).

  3.   If list_len is non-zero, read in the list following
       the header (2 * list_len bytes).

  4.   Read in the message itself (length bytes).

  5.   Process the message.

  6.   If not the end of file, go to step 2.

  To ensure the integrity of the mail file, an initial pass
  over it should be done.  This pass would step through
  each packet in the file, reading each header and making
  sure no packets are truncated.  If the file ends in the
  middle of a packet, then it is obviously corrupted and
  cannot be processed properly.  At this point, either
  throw away the whole file or remove the truncated packet
  and process the remaining packets.

  During the packet tossing, each packet needs to be marked
  as processed.  Thus, if analysis is interrupted before
  completion, the packet analyzer can skip over those
  packets already processed when run again.  To mark the
  packet as already processed (or deleted), set the
  main_type to 65535.  Any packet with a main_type of 65535
  should therefore not be processed.

  The analyzer should follow these steps when processing
  each packet:

  1.   If main_type is 65535 (deleted), skip the message.

  2.   If list_len is non-zero, do steps 3-6 for each entry
       in the list, substituting that entry for "tosys"

                            -16-
 



  3.   If tosys is this system, process the file locally
       (in the case of NETWORK1.EXE, the message is copied
       to LOCAL.NET for processing by the local packet
       processor).

  4.   If tosys is an unknown system or unreachable, put
       the packet in the dead mail file.

  5.   If the next system to forward the packet to ("next
       hop") is the system that the message was last
       received from, put the packet in the dead mail file.
       This prevents messages from looping

  6.   If the tosys is okay, put the packet into the file
       that is to be sent to the next hop.

  Of course, it is more complicated than that.  If list_len
  is non-zero, all destination systems should be processed
  before the message is stored anywhere.  This way, if the
  message has 10 destinations, each of which has the same
  next hop, only one copy of the message will be printed
  out, that packet with 10 systems in its destination list.
  Likewise, for a system with more than one connection, if
  a message has 4 destination systems going through one
  next hop and 3 going through another, one copy of the
  message will go into each hop's file -- one with four
  systems in the node list, the other with the remaining
  three.

  The dead mail file is reprocessed whenever a network
  update (new BBSlist or connection list) is received.

 C.   Local Mail Processing and DEmmm.EXE

  Processing of local mail packets should be similar to
  processing of incoming netmail packets.  The main
  difference between netmail tossing and local mail tossing
  is that the main and minor types are ignored during
  netmail processing, and the list_len and node list are
  ignored (since there won't be a list on local mail).

  1.   Open the file, and set the current message pointer
       to the beginning of the file.

  2.   Read in the net header (24 bytes).

  3.   Read in the message itself (length bytes).

                            -17-
 



  4.   Process the message.  This is done according to the
       main_type and (if applicable) minor_type of the
       message.

  5.   If not the end of file, go to step 2.

  A few main_types (noted below) cause return messages to
  be generated to go back out to other systems.  The local
  mail file analyzer should place these into an interim
  mail file that will be processed by the netmail file
  analyzer after local mail processing has been completed.
  The NETXX local mail analyzer (NETWORK2.EXE) puts these
  messages into P1.001 or P2.001.  Then NETWORK1.EXE
  analyzes this file and places them into outgoing netmail
  files for the system's connections.

  Some types of messages that contain network related files
  such as updates to the nodelists and connect lists and
  email to all sysops from the NC or GC.  For the sake of
  network security, these messages have a
  source-verification signature at the beginning (using,
  for example, RSA or a similar algorithm). There are
  several "methods" used to handle these messages, and each
  requires a decryption program, or DEmmm.EXE (where "mmm"
  is the encryption method, as listed in the 'method. field
  of the net header).  These DEmmm.EXE files MUST be
  supplied by the NC or GC of each network, and each
  network's DEmmm.EXE are unique to that network.  That is,
  NETXX's DE1.EXE will not handle method 1 messages from
  WWIVLink, nor vice versa.

  When a message is encountered with 'method!=0', the
  following steps are taken:

  1.   The local packet analyzer writes out the text of the
       message (no header or node list) to a temporary file
       (TEMPDE.XXX) in the data directory for the relevant
       network.

  2.   A command line for calling the appropriate DEmmm.EXE
       is created using the C command "sprintf(cmd, "de%u
       %s/tempde.xxx", nh.method, net_data);" ('nh' is a
       structure of type net_header_record, 'net_data' is
       the network data directory).  The command is then
       executed.

  3.   The DEmmm.EXE program is then responsible for

                            -18-
 


       reading the TEMPDE.XXX in from disk, deleting it,
       then attempting to decode it.  Two things can then
       happen: a.   If the TEMPDE.XXX fails decoding (bad
       CRC), DEmmm.EXE just exits (returning to the local
       packet analyzer). If the analyzer finds the
       TEMPDE.XXX file does not exist, the message is
       deleted and the program goes to the next packet. b.
       If the CRC checks out in the DEmmm.EXE program, it
       writes out the decoded text into a new TEMPDE.XXX
       file and exits.  The local packet analyzer reads in
       the data from that file and replaces the text of the
       message with that, then goes ahead and processes the
       packet as determined by main_type.

  Network operators who wish to write EN/DE programs for
  their own netwide messages may wish to consider using the
  RSAREF library to develop their own source-verification
  scheme.

 D.   Main and Minor Message Types

  The main and minor type of each message determines how it
  is processed (where it goes, whether it's a sub post,
  email, or network info, etc.).  The analyzer reads the
  message header in first to get the main and minor types,
  then performs the operation indicated by those fields.

  Here is a list of the main_ and minor_types, along with
  the WWIV BBS #define name and descriptions of the
  processing required. Encoding method is assumed to be
  zero unless otherwise noted. Note that while these
  descriptions concern analyzing local mail packets, they
  also apply to how to create outgoing netmail packets.

  main_type 1 (0x0001) -- main_type_net_info

       These messages contain various network information
       files, encoded with method 1 (requiring DE1.EXE).
       Once DE1.EXE has verified the source and returned to
       the analyzer, the file is created in the network's
       DATA directory with the filename determined by the
       minor_type (except minor_type 1).

        0 -- Feedback to sysop from the NC.  This is sent
       to the #1 account as source-verified email.

        1 -- BBSLIST.NET -- Old-style node list (non-Group
       setup).

                            -19-
 



        2 -- CONNECT.NET -- Old-style connections list
       (non-Group).

        3 -- SUBS.LST -- List of subboards ("subs")
       available through the network.  This has been
       replaced by main_type 9.

        4 -- WWIVNEWS.NET -- An electronic magazine of
       sorts distributed within some networks, usually
       monthly.

        5 -- FBACKHDR.NET -- a header file added to network
       update reports for the network.

        6 -- Additional WWIVNEWS.NET text -- appended to
       the existing WWIVNEWS.NET file.

        7 -- CATEG.NET -- List of sub categories.  WWIV's
       sub setup facility uses this list so the sysop can
       specify what category a netted sub falls into.  The
       network's SUBS.LST compiler uses this information
       for compiling the subs lists sent out as main_type
       9.

       8 -- NETWORKS.LST -- A list of all current NETXX
       type networks.

       This message type is source-verified.  That is,
       there is a digital signature at the beginning of the
       message text which is decoded by the DE1.EXE to
       verify that it has come from a "legal" source.  This
       helps make sure that the network info will only come
       from the Network Coordinator.  If the source-
       verification fails, the packet is discarded.

  main_type 2 (0x0002) -- main_type_email

       This is regular email sent to a user number at this
       system (see tosys and touser, above).  Note that
       this type of mail should only be received by systems
       that assign numbers to users (WWIV, VBBS, etc.).
       BBSes in a WWIV network that only identify users by
       name (PCBoard, RBBS, etc.) can only receive
       email-by-name (see main_type 7, below).  Email has
       no minor type, so minor_type will always be zero.

       Processing of the email is straightforward.  The

                            -20-
 


       analyzer looks for the user number indicated by the
       touser field.  If the number exists and the user has
       not been deleted from the BBS, the message is
       written into the email file, addressed to the user
       indicated.  If the number does not exist or the user
       at that number has been deleted, the packet is
       deleted without processing.   Alternatively, the
       analyzer may generate a return message (as email) to
       the sender telling him that the mail was not
       delivered and quoting the message back to him.

  main_type 3 (0x0003) -- main_type_post

       This is a post sent from the sub host's system to
       the subscriber systems, for subs that have a numeric
       sub-type (subs of alphanumeric subtypes are
       main_type 26, described below).  The minor_type will
       be the numeric subtype the post will go to.

       When this type is encountered, the network analyzer
       should search the BBS data files for the sub type
       given.  When the sub is found, the message is
       written into the indicated message file in whatever
       format the BBS software uses.  If the sub type is
       not found, the message packet is simply deleted.
       (There are some local mail preprocessors which will
       scan the packet for messages on subs that the system
       does not carry, and return the message to the host
       system. An alternate mail analyzer could have such a
       capability built in.)

  main_type 4 (0x0004) -- UNUSED

  main_type 5 (0x0005) -- main_type_pre_post

       These messages are similar to main_type_3, except
       they are posts en route from the subscribers to the
       host of a sub. Like main_type 3, the minor_type is
       the numeric subtype of the sub.  Since this is from
       subscriber to host, list_len will be zero.

       When this type of packet is received, the local mail
       tosser should look for the appropriate file which
       will contain the list of subscribers to the sub
       (WWIV and NETxx use N[subtype].NET)  If the file is
       found, a main_type 3 copy of the post is generated
       in an outgoing netmail file, with the node list read
       from the subscriber file inserted before the message

                            -21-
 


       text (and the list_len field modified reflecting the
       addition of the node list).  If the file cannot be
       found, the analyzer assumes the BBS does not host
       the sub and deletes the packet.

       Many WWIV sysops use a feature of the software known
       as network validation ("netval").  When a sub is set
       for netval (this is found in the SUBS.DAT record for
       the sub), the analyzer does not send the post out to
       the network.  The sysop must validate the post on
       the BBS, at which point it is sent out by the BBS.
       This also applies to pre-posts for main_type 26
       (main_type_new_post).

  main_type 6 (0x0006) -- main_type_external

       This type has largely been replaced with main_type
       27 (main_type_new_external), but essentially works
       the same way.  This will create messages that are
       read and processed by an external processor.  The
       minor_type is determined by the program expected to
       work with it.

       When the processor encounters this type of message,
       it searches for a file that contains the names of
       external programs, and the minor_types they accept,
       used by the BBS (EXT_LIST.NET for the NETXX
       software).  If found, the message is written or
       appended to EXTERNAL.NET in the network's data
       directory.  The external program, when run, should
       be able to find the file and process it, then delete
       the file (or the portion that it uses).  Note that
       when there is more than one main_type 6 message in a
       mail file, the EXTERNAL.NET will contain all
       messages of that type, so the external message
       processor needs to be able to find the relevant text
       within the file.

       It is encouraged that programs that use external
       messages use main_type 27 (main_type_new_external),
       which has more robust features.  Among other things,
       that type will create a separate temporary file for
       each main_type 27 message found, making processing
       of external messages simpler.

  main_type 7 (0x0007) -- main_type_email_name

       The other email type.  The touser field is zero, and

                            -22-
 


       the name is found at the beginning of the message
       text, followed by a NUL character.  Minor_type will
       always be zero.

       When this type of packet is encountered, the
       analyzer gets the name from the beginning of the
       message text and searches through the BBS's user
       list to find the specified name.  If it is found,
       the message is written into the email file for the
       BBS.  If it is not, the message will be deleted or a
       return email may be sent back to the sender,
       explaining that the message was not delivered and
       quoting the email back.

       The destination user's name is prepended to the
       beginning of the message text, followed by a NUL,
       then the rest of the usual message header info and
       the message text.  The message header info at the
       beginning of email-by-name messages is thus
       DEST_USER_NAME<nul>TITLE<nul>SENDER_NAME<cr/lf>
       DATE_STRING<cr/lf>MESSAGE_TEXT.

  main_type 8 (0x0008) -- main_type_net_edit

       This type is used by Black Dragon (#1@1180 NETXX)
       in conjunction with his program NETEDIT, a utility
       for managing the network files on a WWIV BBS.  For
       more information on this type, contact him at the
       above address.

  main_type 9 (0x0009) -- main_type_sub_lst

       Networks with large subs lists (over 32k) break them
       up into parts and send the set out under this main
       type rather than the subs.lst type under main_type
       1.  The minor_type indicates the part of the subs
       list.

       When the analyzer processes this type, it creates a
       file in the appropriate network directory and copies
       the message text into it.  Minor_type zero creates
       the file SUBS.LST, and all other minor_types create
       a SUBS.x file where 'x' is the minor_type.

  main_type 10 (0x000a) -- UNUSED

  main_type 11 (0x000b) -- main_type_bbslist


                            -23-
 


       This type is for the new-style BBSLIST files used in
       networks that use the Group setup.  It covers full
       and partial updates sent from the Network
       Coordinator (NC) to the entire network as well as
       update requests sent from the Group Coordinators
       (GCs) to the NC.  The encoding method is either 1
       (when coming from the NC) or calculated as
       ((minor_type % 256)+256) (when coming from the GC).
       Messages of this type are source verified by
       DE<method>.EXE, handled the same as main_type 1
       packets are.  Minor types are as follows:

       0..255    Full bbslist update sent from NC to the
                 network. Minor type is the Group number.
                 Creates BBSLIST.<minor_type> in the
                 network data direc tory.

       257..511  Full bbslist update sent from the GC to
                 the NC. Minor_type is the Group number
                 plus 256.  Creates
                 BBSLIST.A<minor-less-256-in-hex> in the
                 NC's network data directory.

       513..767  Partial bbslist update sent from the NC to
                 the network.  Minor type is the Group
                 number plus 512. Creates the file
                 BBSLIST.<minor-type> in the network data
                 directory.  This file will be merged with
                 the appropriate full BBSLIST file during
                 network analysis (described below).

       In some networks, the Group updates are sent out to
       the network by the GCs rather than the NC.

  main_type 12 (0x000c) -- main_type_connect

       This is the same as main_type 11, except it is for
       CONNECT files.  It also does not include partial
       updates, as there are none for CONNECT files.  The
       encoding method is also either 1 (from NC) or
       ((minor_type % 256)+256) (from NC) for this type.
       These packets are also source-verified, checked by
       DE<method>.EXE.  Minor types:

       0..255    Full CONNECT update sent from NC to the
                 network. Minor type is the Group number.
                 Creates CONNECT.<minor-type> in the
                 network data directory (after

                            -24-
 


                 source-verification).

       257..511  Full bbslist update sent from the GC to
                 the NC. Minor_type is the Group number
                 plus 256.  Creates
                 CONNECT.A<minor-less-256-in-hex> in the
                 NC's network data directory (after source
                 verifica tion).

       In some networks, the Group updates are sent out to
       the network by the GCs rather than the NC.

  main_type 13 (0x000d) -- UNUSED

  main_type 14 (0x000e) -- main_type_group_info

       For now, this type only covers email to the members
       of a particular Group by the GC, so minor type will
       always be zero.  Encoding method is the Group number
       plus 256. Processing for this is the same as for
       type 1/0 messages. They are source-verified by
       DE<method>.EXE.

  main_type 15 (0x000f) -- main_type_ssm

       WWIV BBSes also send out small messages, known as
       "SSMs", across the network.  These are little
       one-line messages generally telling users when their
       mail has been read and so forth.  Some BBSes have
       been modified to send other types of messages.  At
       any rate, main_type handles these small messages.
       Minor_type is always zero.

       Like email, the analyzer searches for the user
       number in the BBS's user list.  If found, the
       message is written into the SSM file for the BBS.
       Since this is a feature most non-WWIV systems do not
       support, they can be ignored.

  One feature of WWIV networking is the ability fpr network
  sysops to send "REQs" to the hosts of subs, enabling them
  to automati cally subscribe to and drop subs they belong
  to.  Main_types 16 through 19 handle REQs and their
  responses.

  main_type 16 (0x0010) -- main_type_sub_add_req

       This is for requests to add the sending system to a

                            -25-
 


       sub's subscriber list.  Minor_type will be the
       numeric sub type, or zero for alphanumeric sub
       types.  For minor_type==0, the sub type (followed by
       NUL) will be the message text. Otherwise, there
       should be no message text (if there is, ignore it).

       When this message type is received, the analyzer
       should search for the subtype's subscriber file
       (N[subtype].NET for WWIV systems).  If the file is
       found, the system number of the new subscriber is
       added, if it is not in there already. If the add is
       successful, it will then look for a text file in the
       data directory (SA[subtype].NET for the NETXX
       software) that contains information the sysop wants
       to send back to the subscriber.   A type 18 message
       is sent to the subscriber indicating status of the
       add request (see below) and including the text in
       the SA[subtype].NET file, if one is found.  If the
       add is not allowed, the analyzer looks for
       SR[subtype].NET and sends that text back to the
       requester. If the add is otherwise unsuccessful, the
       type 18 message will include the status and a short
       explanation of why the add was not succcessful.

  main_type 17 (0x0011) -- main_type_sub_drop_req

       This is the same as a main_type 16 message, except
       that it is sent by a subscriber wishing to drop a
       sub.  The minor_type is determined the same way as
       main_type 16. There should be no message text, but
       if there is any, it is ignored.

       Processing is similar to a type 16 message.  The
       analyzer searches for the subtype's subscriber file,
       and upon finding it removes the subscriber's system
       number from it, if it is there.  Status of the drop
       request is returned to the sender in a main_type 19
       message, with a short message appended that explains
       the status.

  main_type 18 (0x0012) -- main_type_sub_add_resp

       This carries the status response to a main_type 16
       (sub add request).  The minor_type is determined the
       same way as type 16 message.  The first byte of
       message text (after the subtype, if minor_type==0)
       is the status of the add request. Possible status
       byte values are:

                            -26-
 



       0 -- Subscriber added successfully.

       1 -- This system is not the host (N[subtype].NET not
       found).

       3 -- Not allowed to add subscribers automatically.

       4 -- System number already there (can't add it
       again).


       When received, the processor will send the message
       text mentioned in the main_type 16 description
       (minus the status byte) to the sysop as email.

  main_type 19 (0x0013)- main_type_sub_drop_resp

       Similar to main_type 18, this carries the response
       to a sub drop request, with the minor_type following
       the same conventions as above.  This also carries a
       status byte as the message text:

       0 -- Sub dropped successfully.

       1 -- This system is not the host.

       2 -- System number is not there (can't drop it).

       3 - not allowed to drop subscribers automatically.

       When received, the processor will send the message
       text mentioned in ther main_type 17 description
       (minus the status byte) to the sysop as email.

  main_type 20 (0x0014) -- main_type_sub_list_info

       In many WWIV networks, the subs list coordinator
       (SLC) occasionally sends out "pings" to all network
       members.

       When this message type is received from the SLC
       (minor_type 0), the analyzer checks the BBS's sub
       data file.  If there are any subs hosted by the
       receiving system which are flagged for inclusion in
       the distributed SUBS.LST, a list of them is returned
       to the SLC via another main_type 20 message with
       minor_type==1).  When the SLC recieves the reply, it

                            -27-
 


       is written into SUBS.INF in the network's data
       directory (appended if the file exists).

  main_type 26 (0x001a) -- main_type_new_post

       Because of the growing number of networked subs on
       NETXX, the number of available subtypes was
       getting scarce. Starting with WWIV version 4.22 and
       NET32, alphanumeric subtype support was added,
       greatly expanding the possible subtypes.  Alpha
       subtypes are seven characters -- the first must be a
       letter, but the rest can be any character allowed in
       a DOS filename.  This main_type covers both
       subscriber- to-host and host-to-subscriber messages.
       Minor type is always zero (since it's ignored), and
       the subtype appears as the first part of the message
       text, followed by a NUL. Thus, the message header
       info at the beginning of the message text is in the
       format SUBTYPE<nul>TITLE<nul>
       SENDER_NAME<cr/lf>DATE_STRING<cr/lf>MESSAGE_TEXT.

       It is assumed that a message coming into a host is a
       prepost, and it is processed similarly to main_type
       5. Likewise, it is assumed that messages coming into
       a sub scriber system are net posts, and they are
       processed similarly to main_type 3.

  main_type 27 (0x001b) -- main_type_new_external

       This is the new type of external message,
       implemented with NET33.  Like main_type 6, it
       creates an external file with the message text for
       an external program to process.  Again, the
       minor_type is determined by the external program.

       There is a full explanation of how these messages
       are processed in Filo's NETXX Software Docs.  In
       short, similar to main_type 6, the local mail
       processor searches for the minor_type in a data file
       (EPROGS.NET for NETxx), and creates an external file
       if it is found.  When the local mail processor is
       finished with the local mail file, the program
       associated with that minor_type will execute, with
       the associated filename (with path) as a parameter.

V.  BBSLIST/CONNECT FILES AND MESSAGE ROUTING

So how does the network software know where to send an

                            -28-
 


off-system message, especially if the BBS has more than one
connect?  FidoNet has its nodelists, and so does NETXX.
NETXX actually has two different types of node lists, as
mentioned elsewhere.  We'll take these separately, then
discuss figuring out the routing.

 A.  Old NETXX -- BBSLIST.NET & CONNECT.NET

 In the beginning of NETXX, there were only two files
 needed to keep up with the systems in the network --
 BBSLIST.NET and CONNECT.NET.  Though this is rarely used
 now, there are still some smaller networks which use these
 files, so they should be discussed.

 BBSLIST.NET holds a listing of what systems are in the
 network. Each system has an entry, with the systems
 usually grouped by area code.  The format for each
 system's line: system number (preceded by @), system phone
 number (preceded by *), max bps rate of the system
 (preceded by #), system flags if any, WWIV registration
 number or date of entry (enclosed in brackets), and system
 name (enclosed in "").  For example, the BBSLIST line for
 Amber in NETXX could be:

 @1     *310-798-9993  #14400 <  !$     [1]  "Amber"

 Most of the system flags after the modem speed indicate
 the kind of high-speed modem being used by the system.
 Currently, these flags are:

 | -- Telebit-compatible (PEP) modem.
 < -- USR HST 9600+bps modem.
 > -- Hayes V-Series compatible 9600+bps modem.
 Z -- Zoom V.32terbo (19.2kpbs) modem.
 / -- CompuCom 9600+bps modem.
 ! -- CCITT V.32 (9600bps) modem.
 $ -- CCITT V.32bis (14.4kbps) modem.
 ~ -- V.FAST (28.8kbps) modem.
 ? -- Fax-capable modem (not currently used)

 Other system flags used which are not modem designators:

 + -- The system is a dedicated mail server.  That is, it
      is not a true BBS, only handles the transfer of
      network mail for an area or region.

 \ -- Fidonet system.  Some systems in the network have
      "gateways" into Fidonet (or Fidonet compatible, such

                            -29-
 


      as GlobalNet).

 = -- PCPursuitable system.  This is actually not useful
      since PCPursuit has gone out of business (though
      there are other similar networks still operating).

 _ -- End node.  That is, a system with only one
      connection.

 There can also be one of three flags appearing before the
 phone number: ^ -- Area Code coordinator (AC). & --
 Network Coordinator (NC). % -- Group Coordinator (GC) Note
 that since there can only be one Network Coordinator, the
 "&" should only appear once in the BBSLIST.NET file. Also,
 the "%" is not likely to be seen except in the Group setup
 described below, since this setup has no Groups.

 The first line of the BBSLIST.NET must be a tilde (~)
 followed by the Unix timestamp (seconds since midnight,
 Jan 1, 1970) indicating the date and time the file was
 sent out by the NC or GC.

 CONNECT.NET lists the connection costs between systems.
 The cost listed should be the cost per minute, though for
 most networks using this system, the rule of thumb is 0.00
 for local connects, 0.01 for long distance connects, and
 more for long distance connects that one wants to route
 less mail through.

 Each entry in the CONNECT.NET file specifies a one-way
 connection between two systems.  The entries in the
 CONNECT.NET file do NOT need to be in any specific order.
 The format for system's connection entry is: the system
 number (preceded by "@"), first connection and cost
 (separated by "="), second connection and cost, and so
 forth.  Like BBSLIST.NET, the first line is a tilde (~)
 followed by the UNIX timestamp.

 Examples:
 1.   If there are two systems, numbered 1 and 2, and each
      can call each other for free, the CONNECT.NET file
      would look like:

      @1      2=0.00
      @2      1=0.00

      Note that the routing analysis software should make
      sure both ends of the connection have entries

                            -30-
 


      referring to each other.

 2.   If there are three systems, each can call the others
      for free, the CONNECT.NET file would look like:

      @1      2=0.00 3=0.00
      @2      1=0.00 3=0.00
      @3      1=0.00 2=0.00

 3.   If system 3 called the other two for $0.10, the
      CONNECT.NET file would look like:

      @1      2=0.00 3=0.10
      @2      1=0.00 3=0.10
      @3      1=0.10 2=0.10


 In both BBSLIST.NET and CONNECT.NET file, each entry
 begins with identifying the system number (preceded with
 @), allowing system entries to take up more than one line
 -- in larger networks a CONNECT.NET entry can fill more
 than one line.  Everything after the system number
 identifier up to the next "@", corresponds to that system.
 Thus, the CONNECT.NET files above could also be listed as:

 @1 2=0.00 3=0.10 @2 1=0.00 3=0.10 @3 1=0.10 2=0.10

 or

 @1
 2=0.00
 3=0.10
 @2
 1=0.00
 2=0.10
 @3
 1=0.10
 2=0.10

 Thus, the end-of-line indicator (EOL) should be IGNORED.

 Neither the BBSLIST.NET nor CONNECT.NET file need to be in
 any specific order.  There cannot, however, be multiple
 entries per system in either BBSLIST.NET or CONNECT.NET.

 It is possible for a system to have references in one or
 both of the .NET files, but not be reachable from any
 other system.  For example, two systems may be listed in

                            -31-
 


 BBSLIST.NET, and listed in CONNECT.NET, but each can only
 call the other.  No other system in the network can
 connect with them, so they don't exist, essentially.
 Also, a system can be listed in BBSLIST.NET, but not have
 any entries in CONNECT.NET.  This usually happens for
 systems just joining the network, and those systems
 essentially don't exist either.

 It is also possible for one system to call another, but
 the second system can't call back the first.  This is
 unusual, but valid.  Also, the cost of a connection can be
 different in one direction than it is in the other.  This
 is also valid.

 BBSLIST.NET is received across the network as main_type 1,
 minor_type 1 (1/1).  CONNECT.NET is received as main_type
 1, minor_type 2 (1/2).

 B.   New NETXX -- BBSLIST.x and CONNECT.x

  As mentioned previously, things changed in 1990.  The
  original NETXX network had grown so large that it was
  necessary to break the BBSLIST and CONNECT files into
  smaller segments, known as Groups.  How the Groups are
  determined is up to the Network Coordinator.  NETXX's
  Groups were formed based on the connec tion topology at
  the time of the conversion to the Group system. Many
  other networks use the International Time Zones to divide
  the groups.  The Groups are numbered, with the potential
  for up to 255 Groups in a network.  The BBSLIST and
  CONNECT files have the Group number ("x") as the
  extension.

  The BBSLIST.x file is formatted the same way as the
  BBSLIST.NET under the old NETXX system.  Only BBSes
  with the Group are in each Group's files.  There is an
  additional file, BBSLIST.0, which contains information
  for the routing analyzer.  The first line has the UNIX
  timestamp, as usual.  The second describes which
  CONNECT.x file to use.  If it is ":" alone, then
  CONNECT.0 contains all the connection information for the
  network, so that is all that will be used.  If the second
  line is ":A", then CONNECT.0 and the CONNECT.x files for
  all Groups are used.

  There can also be partial BBSLIST updates sent out,
  indicating systems to be added, changed or removed.  The
  extension on these messages is generally Group number

                            -32-
 


  plus 512 (e.g., a group 1 partial update would be
  BBSLIST.513).  For added or changed systems, the system
  is listed as it would appear in the BBSLIST.x it belongs
  to.  A deleted system is listed as just the system number
  with a period (.), as in "@1234 .".  These partial
  updates are incorporated into the full BBSLIST files
  during the routing analysis (see below).

  The BBSLIST.x files use the same indicators as the
  BBSLIST.NET file, with one addition: This should only
  appear once in each BBSLIST.x file, since each Group may
  only have one GC.

  CONNECT.x is not like the old CONNECT.NET.  The main
  difference is that there are no costs in the connections,
  only the node numbers each system is connected to.  So in
  the second example in the previous section, the CONNECT.x
  for systems 1, 2 and 3 would look like this:

  @1   2 3
  @2   1 3
  @3   1 2

  As noted above, the CONNECT.x files will be used as
  specified by the second line in BBSLIST.0.  When that
  line is ":", CONNECT.0 will contain all the network's
  connections.  When it is ":A", the CONNECT.x files
  contain all the connections within each Group. All
  connections between systems in different groups are
  listed in CONNECT.0.

  For example, say we have a network with five systems,
  numbered 1, 2, 3, 4, and 5.  Systems 1 and 2 are in Group
  1, and systems 3, 4, and 5 are in Group 2.  1 and 2
  connect to each other, and 2 connects to 4.  3 and 4
  connect to each other, and 5 connects to 1.

  CONNECT.1 will contain:
  @1   2
  @2   1

  CONNECT.2 will contain:
  @3   4
  @4   3

  CONNECT.0 will contain:
  @1   5
  @2   4

                            -33-
 


  @4   2
  @5   1

  Like the old NETXX files, the ordering of the systems
  in these files does not matter, however a node number may
  appear only once in all of the BBSLIST.x files combined.

  The BBSLIST.x files are received across the networks as
  main_type 11, with the minor_type being determined by the
  Group they are for.  CONNECT.x files are received as
  main_type 12, with the minor_type determined by the Group
  number.

 C.  Figuring the Routing

  There are three ways message routing could be determined:

  1.   Each time you need to route a message, find the
       least cost or shortest path (depending on whether
       new or old update files are being used);

  2.   Each time one of the network programs that has to
       route messages is run, the least-cost or shortest
       route to each system is decided; or

  3.   Each time an update to the .NET files is received,
       the least cost route is decided to each system.

  Options 1 and 2 are simply not practical.  Depending on
  network size and system speed, it can take a minute or
  more to analyze the network data files and determine the
  optimal route.  Finding the best route to a specific
  system requires the same operation as finding the best
  route to all systems, so option #1 is a waste of time
  (besides possibly requiring the BBS to have the path-
  finding code in it).  Option #2 holds no advantages over
  option #3 because it will tie up the BBS unnecessarily.

  Therefore, the optimal routes to all the systems in the
  network should be analyzed only when a network update is
  received. This routing analysis can be done any way, as
  long as it determines the best route.  The best way,
  however, could follow these steps:

  1.   System records are read into an array from
       BBSLIST.NET (for the old NETXX setup) or the
       BBSLIST.x files (for the Group setup).  The array is
       of struct net_system_list_rec (see below).  All

                            -34-
 


       fields are filled from the BBSLIST.x except numhops
       and forsys, which are set to 65535 and 0, respec
       tively.  For the Group setup, the BBSLIST.xxx
       partial updates (.513-.768) are read in next, and
       the indicated changes (existing systems replaced,
       new systems added, systems indicated with "."
       deleted) are made in the array and in the BBSLIST
       files themselves.  After processing, the partial
       updates are deleted.

  2.   Next, the CONNECT files are read into another array.
       Since system records may be repeated in the CONNECT
       files, make sure, when each system record is read
       in, a check is made in the array to see if there is
       already an entry for it.  If there is, add the
       connections in the new record to the existing
       record.

  3.   Then, the analysis starts.  The analyzer uses the
       system's callout list (CALLOUT.NET for NETxx) as a
       base, starting with the first entry and checking
       each of its connnects, spreading out from there to
       THEIR connects.  This is done for each system in the
       callout list.

       For each destination system checked, the number of
       hops found is compared to that entered in the
       network data array (numhops) and changed if it is
       less.  The forsys is also changed, if that is
       different.  This is for the Group setup.

       If the network is using the old NETXX setup, cost,
       rather than number of hops, is considered.  In this
       case, when figuring cost, the speed of a connection
       (highest speed two connecting systems will support)
       needs to be considered. For instance, two systems
       connecting at 14400bps at a cost of 0.10 would take
       precedence over two systems connectiung at 2400bps
       at a cost of 0.10 (assuming that they are on the
       path to the same destination system, and all other
       scosts are equal).

  4.   When all systems in the callout list are processed,
       analysis is complete and the network data array is
       written to disk. If specified, a piece of mail is
       then sent to the sysop giving the results of the
       analysis (for instance, how many valid systems are
       in the network, how many systems route through each

                            -35-
 


       of this system's connections, who the NC and GC and
       AC are, and so forth).

  The data structure NETxx's NETWORK3.EXE uses for the
  network data file is:

  typedef struct {
       unsigned short sysnum;
             /* system number */
       char           phone[13]
       ,     /* phone number of system */
                      name[40];
             /* name of system */
       unsigned char  group;
             /* group system is in */
       unsigned short speed,
             /* max bps rate of system */
                      other,
             /* other info (bit mapped) */
                      forsys;
             /* next hop from here */
       short          numhops;
             /* how far to get there */
       union {
            unsigned short rout_fact;
             /* routing factor */
            float          cost;
            /* cost factor */
            long           temp;
            /* temporary variable */
       } xx;
  } net_system_list_rec;

  It is encouraged that this structure be used by any
  NETXX compatible analyzer.  Not only is this used by
  the WWIV BBS software, but some NETXX add-ons also use
  this file, so supporting this structure will enhance
  compatibility with NETXX.

  The fields: sysnum, phone, name, group, and speed should
  be self-explanatory.

  other --  This is bitmapped, and contains the modem and
            other information shown in BBSLIST.  The bitmap
            values are (with corresponding BBSLIST flag):

            \    other_fido          0x0001
            |    other_Telebit_19200 0x0002

                            -36-
 


            <    other_USR_9600      0x0004
            >    other_Hayes_9600    0x0008
            ^    other_coordinator   0x0010
                 (area coordinator)
            !    other_V32           0x0020
            $    other_V32bis        0x0040
            =    other_PCP           0x0080
            %    other_group_coord   0x0100
            &    other_net_coord     0x0200
            /    other_compucom      0x0400
            +    other_net_server    0x0800
            ?    other_FAX           0x1000
            _    other_end_system    0x2000
            ~    other_VFAST         0x4000

  forsys -- Where to forward messages destined for this
            system, also known as "next hop".  For example,
            if a message going from system 1 to system 5
            passes through systems 2 and 4, then forsys==2.
            When it is determined that the system is
            unreachable (listed in BBSLIST but no
            connections listed), forsys==65535.

  rout_fact -- This is the routing factor, but is currently
            not used by the NETxx software.

  cost --   When using an old-style WWIV network setup,
            this holds the cost of the call, calculated as
            the sum of costs for each hop to the
            destination.

  When all systems have been processed, you should have a
  database containing all systems in the network and how
  they may be reached.  Whenever a packet that is not
  destined for the local system is processed, the data file
  is searched to find the system entry for the destination
  system.  If it is not found, then the system is unknown.
  If the system is identified as unreachable
  (forsys==65535), the system is also considered unknown.

VI.  TIPS FOR WRITING WWIVNET SOFTWARE

 That about covers all the technical details for designing
 software compatible with WWIV networks.  Now for some
 things to consider for those wishing to design a NETXX
 interface for a non-WWIV BBS, or add-ons to existing
 NETXX software.


                            -37-
 


 A.   NETXX Interface Software

  The information provided in this document is enough for
  anyone wishing to write NETXX interface software from
  scratch.  Unless you are writing for a BBS on a non-PC
  platform (such as Hermes for Macintosh), there is no need
  to rewrite all of the software to interface a PC-based
  BBS to a WWIV network.  Since the local mail processor
  (NETWORK2.EXE) is the only program that writes to BBS
  message bases, that is really the only one needing
  replace ment.  If any of the NETxx programs are used, it
  is essential that all of the supporting data files used
  by that software be present.  For details on those files,
  see the NETXX software documentation.

  Some additional programming may be necessary, though.
  For one, a shell would be useful for executing the
  various network programs, unless the BBS can be modified
  to make the calls itself.  A batch file could do it, but
  a program such as Jim Wire's CLOUT makes it much easier.
  Any shell or BBS modification should follow these steps
  (filenames in parentheses are programs from NET34 or
  files created/used by them):

  1.   Choose a system to call (or have one specified),
       then execute the network callout program
       (NETWORK.EXE).  If successful, proceed to step 2.
       If not, either try again or end processing.

  2.   Check for the incoming netmail file (P*.NET).  If
       there isn't one, end processing.  If there is one,
       run the netmail packet analyzer (NETWORK1.EXE).

  3.   Check for the local mail file packet (LOCAL.NET).
       If there is none, end processing.  If there is one,
       run the local mail packet analyzer (NETWORK2.EXE).

  4.   Check again for a netmail packet (outgoing messages
       result ing from local mail processing).  If there is
       one run the netmail analyzer, otherwise proceed to
       the next step.

  5.   Check for a BBSLIST or CONNECT update.  The most
       reliable way to do this is to compare the filedates
       of the CONNECT and BBSLIST files against the
       filedate of the database file created last time the
       routing analyzer ran.  If one or more of the files
       is new (or there is a partial BBSLIST update), run

                            -38-
 


       the routing analyzer (NETWORK3.EXE).

  If the software cannot be modified to handle these steps,
  it is probably best to use a front-end such as Front Door
  or Binkleyterm, then set up events that would run the
  shell for making the network calls and processing.

  The trickiest part is exporting messages from the BBS to
  a WWIV network file.  Possibly the easiest way to pack
  new messages is to have the BBS write them out to Fido
  packets (if the BBS is Fido-compatible), then when
  control returns to the front end, run an event that
  converts the Fido packet to a WWIV mail file.  When doing
  this, keep in mind that WWIV networks do not have all of
  the fields a WWIV packet does, most notably the "To:"
  field.

  Another method could be a program that, after the BBS
  returns control to the front end, scans the BBS's message
  bases for new messages on the NETXX subboards.  This
  would work best for BBS programs that cannot export Fido
  messages.  In either case, it is important that the
  netmail file processor analyze the outgoing message file
  (P*.NET) for tossing into the various connection files
  (S*.NET and Z*.NET).

  Of course, the optimal solution, if possible, would be to
  modify the BBS software to export the messages directly
  into a NETXX compatible mail file, and run the other
  network programs as needed without the shell.

  This is probably be a good time to discuss the naming of
  the incoming and pending netmail files, mentioned in step
  2 above. The actual name of the P*.NET can vary,
  depending on NETxx version and what program generates it.
  NETWORK.EXE in older NETxx versions (NET33 and below)
  receive the netmail file as P1.NET, while the one in
  NET34 receives the file as P1-0-1.NET. The "-0" in the
  middle indicates that NETWORK.EXE created the file (think
  of it as NETWORK0).  When other NET34 programs generate
  pending netmail files, the middle number indicates which
  program created it (NETWORK2.EXE, the local mail
  processor, would create a pending netmail file named
  P1-2-1.NET).  The main reason for this new naming system
  is so that we can tell the source of a P*.NET file being
  processed by the netmail analyzer.

  The WWIV BBS just creates P0.NET (network email,

                            -39-
 


  generally) and P1.NET (outggoing sub posts, generally).
  WWIV 4.23 also creates a PGATE.NET file, which contains
  posts for "gated" subs (that is, subs which are carried
  on more than one network).  WWIV 4.24 does not use
  PGATE.NET for gated messages.  Multi-instance WWIV 4.23
  and above setups create P*.nnn (where 'nnn' is the
  instance number, such as P*.001 for instance 1) while a
  user is online, but they are renamed to P*.NET after the
  user logs off.

  The NETWORK1.EXE processes all of the P*.NET files, until
  none are left, before converting any indicated
  S<sysnum>.NET files into compressed Z<sysnuum>.NET files
  (see Appendix A).  It is important that an alternate
  netmail file analyzer be able to recognize and handle any
  P*.NET file, not just Pn.NET.


 B.   NETXX Software Add-ons

  There are two possible types of add-ons supported by the
  NETXX software, both working with the local mail
  processor.

  The external message processors (or "post-processors")
  are described above in the main_type descriptions.  As
  noted above, it is recommended that any post-processor be
  written to be compatible with main_type 27, because it
  provides an easier interface for external messages.
  Again, a full description of how to use the external
  message feature is provided in the NETXX Software
  Documentation written by Filo.

  A common use for external messages is what is known as a
  "ping," used by the authors of some WWIV network
  utilities who wish to gain some information about the use
  of their software.  The author sends out a main_type 27
  message with the minor_type they are using.  If a
  receiving system is using the software and it is
  installed properly, it will execute after local mail
  processing, and process the request in the external
  message.

  The local mail processor also supports use of
  "pre-processors," generally used to scan the local mail
  file for certain types of messages before the local mail
  processor gets to it.  One example is JAFO's AUTOSEND,
  which looks for sub requests and sends out messages from

                            -40-
 


  the system's subs to new subscribers.  These, also, are
  described in the NETXX Software Documentation.

  Naturally, any external message processor or preprocessor
  that generates new outgoing network messages must put
  them into a P*.NET file so that NETWORK1.EXE can find it
  and process it.

APPENDIX A

MAIL PACKET COMPRESSION

 In order to write NETXX software that can deal with
 compressed mail packets, you must have the PKWare
 Compression Libraries, available from PKWare, Inc. for
 $300.00.  This Appendix covers the necessary details for
 handling compressed mail packets.  To make the explanation
 easier, how NETWORK1 from NET34 handles compressed files
 will be explained.

 When NETWORK1 analyzes the file of messages to go out on
 the network (P*.NET), they are placed in Sxxxx.NET files
 (where xxxx corresponds to the numbers of systems in the
 CALLOUT.NET).  After processing of all P*.NET files,
 NETWORK1 checks to see which connections accept compressed
 files.  For each that does, its Sxxxx.NET is compressed
 with the implode() function from the PKWare libraries.
 The compressed data is appended to the corresponding
 Zxxxx.NET file (which is created if it does not exist).
 The size of the compressed segment in Zxxxx.NET is then
 checked against the size of the Sxxxx file.  If the
 compressed segment is smaller than the original file, the
 file header and segment header (see below) are updated and
 Zxxxx.NET is closed. If the compressed file is the same
 size or larger than the uncompressed file, the
 uncompressed version is appended to the Zxxxx.NET
 (overwrit ing the compressed version), then the headers
 are updated and the file closed.  Whether the original
 Sxxxx.NET was compressed or not, it is deleted after it is
 transferred to the Zxxxx.NET.

 Thus, while an uncompressed netmail file is simply a
 collection of message packets with their headers, the
 compressed netmail file is a collection of segments which
 contain one or more messages, either compressed or not.
 The file has a ten byte header, and each segment within
 the file has a five byte header.


                            -41-
 


 The netmail file header has three elements: compression
 identifier -- long int (4 bytes) Always set to 0xfffefffe.

 extra bytes -- unsigned short int (2 bytes)

      Number of additional bytes in the header record,
      being the sum of the bytes in all fields following .
      This is to allow for future expansion of the header
      while maintaining compatibility with older versions
      of the NETxx software.  Currently, this should have
      the value of 4.

 uncompressed bytes -- long int (4 bytes)

      Length of the file when it is uncompressed.  This
      gets updated each time a new segment is added to the
      compressed file.

 The header on each segment of the compressed file has two
 elements: compression flag -- char (1 byte)

      Set to 0 if segment is NOT compressed, 1 if it is.

 segment length -- long int (4 bytes)

      Set to the actual length of the segment, in bytes.

 When a netmail file is received, NETWORK1 reads the first
 four bytes. If they are 0xfffefffe, it knows the file is
 compressed, so it decompresses it before processing the
 messages.  The first ten bytes are read in order to get
 the uncompressed length of the file.  Then for each
 segment, these steps are followed:

 1.   The segment header is read in, to see if the segment
      is com pressed and how long the segment is.

 2.   If the segment is compressed, it is decompressed into
      a temporary netmail file (which is created for the
      first segment, and appended for each additional
      segment) using PKWare's 'explode()' function.  If it
      is NOT compressed, it is written directly to the
      temporary netmail file.

 Once all segments have been decompressed or written to the
 temporary netmail file, the original netmail file is
 deleted and the temporary netmail file is renamed to the
 original's name.  NETWORK1 then processes message packets

                            -42-
 


 the new uncompressed netmail file.

COMPRESSION SOURCE CODE

 The following code is provided to help simplify the
 process of writing code for complete compatibility with
 the NETXX software.  It is the same as what is used by
 NETWORK1.EXE in NET34.  It covers both the compression and
 decompression of netmail packets.  Comments have been
 added by WH in order to clarify what's happening.  Some
 lines are split due to space.

 /* Description of global variables used here:
  *   (long) nbw --  number of bytes written
  *   (long) nbr --  number of bytes read
  *   (long) nbl --  number of bytes left (to read/write)
  *   (int) fi --    input file handle
                     (set to "S[sysnum].NET")
  *   (int) fo --    output file handle
                     (set to "Z[sysnum].NET" for
                     compression, "TEMP.NET" for
                      decompression)
  *   (char) net_data -- path to system's network data
                         directory
  * The rest should be obvious from their use.
  */


 unsigned far pascal net_read(char far *buff,
                             unsigned short int far *size)
 /* used
 {
   unsigned br=0,sz;
   unsigned pct,i;

   sz=*size;
   if ((long)sz>nbl)
     sz=(unsigned)nbl;

   br=read(fi,buff,sz);

   if (br<0)
     br=0;

   nbr += br;
   nbl -= br;
   nc_sf += br;


                            -43-
 


   return(br);
 }


 void far pascal net_write(char far *buff,
                           unsigned short int far *size)
 {
   write(fo,buff,*size);
   nbw += *size;
 }


 void net_compress(unsigned int sn)
 {
   char s[81], s1[81], fl;
   long l,l1;
   char *buf;
   unsigned short int type, dsize, xx;

   /* set up the input (Sxxxx.NET) and output (Zxxxx.NET)
    filenames */

   sprintf(s,"%sS%u.net",net_data, sn);
   sprintf(s1,"%sZ%u.net",net_data, sn);

   /* open the input file, if possible */
   fi=open(s,O_RDWR | O_BINARY);
   if (fi<0) {
     return;
   }

   buf=malloc(35256);
   if (!buf) {
     printf("\r    Not enough mem to compress    \r");
     return;
   }

   /* open the output file, if there is one */
   /*note the following line with CAPS in it should be
    all one line */
fo=open(s1,O_RDWR | O_BINARY | O_CREAT,
   S_IREAD | S_IWRITE);
   if (fo<0) {
     close(fi);
     free(buf);
     return;
   }
   /* write file header if file is new */

                            -44-
 


   if (filelength(fo)==0) {
     /* compression identifier */
     l=0xfffefffe;
     write(fo,&l,4);
     /* extra bytes in header */
     xx=4;
     write(fo,&xx,2);
     /* uncompressed bytes (initalized to 0) */
     l=0L;
     write(fo,&l,4);
   }

   /* prepare for new segment */
   nbw=nbr=0;
   l=filelength(fo);
   lseek(fo,l,SEEK_SET);
   l1=filelength(fi);
   nbl=l1;
   fl=1;             /* compresssion flag (compressed) */
   /* write compression flag and segment length to segment
    header */
   write(fo,&fl,1);
   write(fo,&nbw,4);
   type=CMP_ASCII;
   if (l1<1024)
     dsize=1024;
   else if (l1<2048)
     dsize=2048;
   else
     dsize=4096;

   /* compress the file */
   implode(net_read, net_write, buf, &type, &dsize);

   if (nbw>=nbr) {
     /* if it didn't compress */
     lseek(fo,l,SEEK_SET);
     lseek(fi,0L,SEEK_SET);
     fl=0;
     /* change segment header (flag off, seg length is
      input length */
     write(fo,&fl,1);
     write(fo,&nbr,4);
     /* then write input file to output file (overwrite
      compressed) */
     xx=read(fi,buf,32768);
     while (xx>0) {
       write(fo,buf,xx);

                            -45-
 


       xx=read(fi,buf,32768);
     }
     chsize(fo,l+5+nbr);
   } else {
     /* if compressed, write compressed seg length to
      segment header */
     lseek(fo,l+1,SEEK_SET);
     write(fo,&nbw,4);
   }

   /* update output file header (change uncompresssed
    bytes) */
   lseek(fo,6,SEEK_SET);
   read(fo,&l,4);
   l += nbr;
   lseek(fo,6,SEEK_SET);
   write(fo,&l,4);

   bytes_comp=filelength(fo);
   bytes_uncomp=l;
   /* compute percentage of compression */
   if (bytes_comp<bytes_uncomp)
xx=(unsigned) ((bytes_uncomp-bytes_comp)*100/bytes_uncomp);
   else
     xx=0;

   /* clean up */
   close(fi);
   close(fo);
   unlink(s);
   free(buf);
 }


 void net_uncompress(char *fn)
 /* 'fn' is the name (with path) of the P*.NET file being
  processed */
 {
   char s[81],fl;
   long l,l1;
   unsigned xx;
   char *buf;

   /* set up output filename (temporary netmail file) */
   sprintf(s,"%sTEMP.NET",net_data);

   buf=malloc(16384);
   if (!buf) {

                            -46-
 


     printf("\r    Not enough mem to uncompress  \r");
     return;
   }

   /* Zxxxx.NET, if possible */
   fi=open(fn,O_RDWR | O_BINARY);
   if (fi<0) {
     free(buf);
     return;
   }

   /* open output file */
fo=open(s,O_RDWR | O_BINARY | O_CREAT | O_TRUNC, S_IREAD |
           S_IWRITE);
   if (fo<0) {
     close(fi);
     free(buf);
     return;
   }

   /* get file header */
   lseek(fi,4,SEEK_SET);            /* compression
                                       identifier */
   read(fi,&xx,2);                  /* extra bytes */
   read(fi,&bytes_uncomp,4);        /* uncompressed
                                            bytes */
   bytes_comp=filelength(fi);
   lseek(fi,6+xx,SEEK_SET);
   l=bytes_comp-(6+xx);             /* compute
                                       compressed bytes */

   /* decompression pass */
   while (l>0) {
     /* get segment header */
     read(fi,&fl,1);                /* compression flag */
     read(fi,&l1,4);                /* segment length (in
                                       bytes) */
     nbr=nbw=0;
     nbl=l1;
     if (fl==0) {
 /* if segment not compressed, write directly to temporary
  * netmail file */
       if (nbl>16384)
         xx=read(fi,buf,16384);
       else
         xx=read(fi,buf,(unsigned)nbl);
       while (nbl>0) {
         write(fo,buf,xx);

                            -47-
 


         nbl -= (long)xx;
         if (nbl>16384)
           xx=read(fi,buf,16384);
         else
           xx=read(fi,buf,(unsigned)nbl);
       }
     } else {
/* if segment compressed, decompress to temp
 * netmail file */
       explode(net_read, net_write, buf);
     }
     l -= (l1+5);
   }

   /* clean up */
   close(fi);
   close(fo);
   unlink(fn);
   rename(s,fn);       /* rename temp filename to P*.NET */
   free(buf);
 }




























                            -48-
```
