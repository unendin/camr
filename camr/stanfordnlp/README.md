# Python interface to Stanford Core NLP tools v1.3.3

This is a Python wrapper for Stanford University's NLP group's Java-based [CoreNLP tools](http://nlp.stanford.edu/software/corenlp.shtml).  It can either be imported as a module or run as a JSON-RPC server. Because it uses many large trained models (requiring 3GB RAM on 64-bit machines and usually a few minutes loading time), most applications will probably want to run it as a server.


   * Python interface to Stanford CoreNLP tools: tagging, phrase-structure parsing, dependency parsing, named entity resolution, and coreference resolution.
   * Runs an JSON-RPC server that wraps the Java server and outputs JSON.
   * Outputs parse trees which can be used by [nltk](http://nltk.googlecode.com/svn/trunk/doc/howto/tree.html).


It requires [pexpect](http://www.noah.org/wiki/pexpect) and (optionally) [unidecode](http://pypi.python.org/pypi/Unidecode) to handle non-ASCII text.  This script includes and uses code from [jsonrpc](http://www.simple-is-better.org/rpc/) and [python-progressbar](http://code.google.com/p/python-progressbar/).

It runs the Stanford CoreNLP jar in a separate process, communicates with the java process using its command-line interface, and makes assumptions about the output of the parser in order to parse it into a Python dict object and transfer it using JSON.  The parser will break if the output changes significantly, but it has been tested on **Core NLP tools version 1.3.3** released 2012-07-09.

## Download and Usage 

To use this program you must [download](http://nlp.stanford.edu/software/corenlp.shtml#Download) and unpack the tgz file containing Stanford's CoreNLP package.  By default, `corenlp.py` looks for the Stanford Core NLP folder as a subdirectory of where the script is being run.

In other words: 

    sudo pip install pexpect unidecode   # unidecode is optional
    git clone git://github.com/dasmith/stanford-corenlp-python.git
	  cd stanford-corenlp-python
    wget http://nlp.stanford.edu/software/stanford-corenlp-[version].tgz
    tar xvfz stanford-corenlp-[version].tgz

Then, to launch a server:

    python corenlp.py

Optionally, you can specify a host or port:

    python corenlp.py -H 0.0.0.0 -p 3456

That will run a public JSON-RPC server on port 3456.

Assuming you are running on port 8080, the code in `client.py` shows an example parse: 

    import jsonrpc
    from simplejson import loads
    server = jsonrpc.ServerProxy(jsonrpc.JsonRpc20(),
            jsonrpc.TransportTcpIp(addr=("127.0.0.1", 8080)))

    result = loads(server.parse("Hello world.  It is so beautiful"))
    print "Result", result

That returns a dictionary containing the keys `sentences` and (when applicable) `corefs`. The key `sentences` contains a list of dictionaries for each sentence, which contain `parsetree`, `text`, `tuples` containing the dependencies, and `words`, containing information about parts of speech, NER, etc:

	{u'sentences': [{u'parsetree': u'(ROOT (S (VP (NP (INTJ (UH Hello)) (NP (NN world)))) (. !)))',
	                 u'text': u'Hello world!',
	                 u'tuples': [[u'dep', u'world', u'Hello'],
	                             [u'root', u'ROOT', u'world']],
	                 u'words': [[u'Hello',
	                             {u'CharacterOffsetBegin': u'0',
	                              u'CharacterOffsetEnd': u'5',
	                              u'Lemma': u'hello',
	                              u'NamedEntityTag': u'O',
	                              u'PartOfSpeech': u'UH'}],
	                            [u'world',
	                             {u'CharacterOffsetBegin': u'6',
	                              u'CharacterOffsetEnd': u'11',
	                              u'Lemma': u'world',
	                              u'NamedEntityTag': u'O',
	                              u'PartOfSpeech': u'NN'}],
	                            [u'!',
	                             {u'CharacterOffsetBegin': u'11',
	                              u'CharacterOffsetEnd': u'12',
	                              u'Lemma': u'!',
	                              u'NamedEntityTag': u'O',
	                              u'PartOfSpeech': u'.'}]]},
	                {u'parsetree': u'(ROOT (S (NP (PRP It)) (VP (VBZ is) (ADJP (RB so) (JJ beautiful))) (. .)))',
	                 u'text': u'It is so beautiful.',
	                 u'tuples': [[u'nsubj', u'beautiful', u'It'],
	                             [u'cop', u'beautiful', u'is'],
	                             [u'advmod', u'beautiful', u'so'],
	                             [u'root', u'ROOT', u'beautiful']],
	                 u'words': [[u'It',
	                             {u'CharacterOffsetBegin': u'14',
	                              u'CharacterOffsetEnd': u'16',
	                              u'Lemma': u'it',
	                              u'NamedEntityTag': u'O',
	                              u'PartOfSpeech': u'PRP'}],
	                            [u'is',
	                             {u'CharacterOffsetBegin': u'17',
	                              u'CharacterOffsetEnd': u'19',
	                              u'Lemma': u'be',
	                              u'NamedEntityTag': u'O',
	                              u'PartOfSpeech': u'VBZ'}],
	                            [u'so',
	                             {u'CharacterOffsetBegin': u'20',
	                              u'CharacterOffsetEnd': u'22',
	                              u'Lemma': u'so',
	                              u'NamedEntityTag': u'O',
	                              u'PartOfSpeech': u'RB'}],
	                            [u'beautiful',
	                             {u'CharacterOffsetBegin': u'23',
	                              u'CharacterOffsetEnd': u'32',
	                              u'Lemma': u'beautiful',
	                              u'NamedEntityTag': u'O',
	                              u'PartOfSpeech': u'JJ'}],
	                            [u'.',
	                             {u'CharacterOffsetBegin': u'32',
	                              u'CharacterOffsetEnd': u'33',
	                              u'Lemma': u'.',
	                              u'NamedEntityTag': u'O',
	                              u'PartOfSpeech': u'.'}]]}],
	u'coref': [[[[u'It', 1, 0, 0, 1], [u'Hello world', 0, 1, 0, 2]]]]}
    
To use it in a regular script or to edit/debug it (because errors via RPC are opaque), load the module instead:

    from corenlp import *
    corenlp = StanfordCoreNLP()  # wait a few minutes...
    corenlp.parse("Parse it")

<!--

## Adding WordNet

Note: wordnet doesn't seem to be supported using this approach.  Looks like you'll need Java.

Download WordNet-3.0 Prolog:  http://wordnetcode.princeton.edu/3.0/WNprolog-3.0.tar.gz
tar xvfz WNprolog-3.0.tar.gz 

-->


## Questions 

**Stanford CoreNLP tools require a large amount of free memory**.  Java 5+ uses about 50% more RAM on 64-bit machines than 32-bit machines.  32-bit machine users can lower the memory requirements by changing `-Xmx3g` to `-Xmx2g` or even less.
If pexpect timesout while loading models, check to make sure you have enough memory and can run the server alone without your kernel killing the java process:

	java -cp stanford-corenlp-2012-07-09.jar:stanford-corenlp-2012-07-06-models.jar:xom.jar:joda-time.jar -Xmx3g edu.stanford.nlp.pipeline.StanfordCoreNLP -props default.properties

You can reach me, Dustin Smith, by sending a message on GitHub or through email (contact information is available [on my webpage](http://web.media.mit.edu/~dustin)).


# Contributors

This is free and open source software and has benefited from the contribution and feedback of others.  Like Stanford's CoreNLP tools, it is covered under the [GNU General Public License v2 +](http://www.gnu.org/licenses/gpl-2.0.html), which in short means that modifications to this program must maintain the same free and open source distribution policy.

This project has benefited from the contributions of:

  * @jcc Justin Cheng 
  * Abhaya Agarwal

## Related Projects

These two projects are python wrappers for the [Stanford Parser](http://nlp.stanford.edu/software/lex-parser.shtml), which includes the Stanford Parser, although the Stanford Parser is another project.
  - [stanford-parser-python](http://projects.csail.mit.edu/spatial/Stanford_Parser) uses [JPype](http://jpype.sourceforge.net/) (interface to JVM)
  - [stanford-parser-jython](http://blog.gnucom.cc/2010/using-the-stanford-parser-with-jython/) uses Python

