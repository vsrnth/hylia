---
date: 2020-09-20
title: Sidekiq titbits
tags:
- sidekiq
- rails
- ruby

---
Advanced Sidekiq API's:

Finding and triggering sidekiq jobs of a particular queue from the rails console:

    queue = Sidekiq::Queue.new("sidekiq_queue")
    
    // if all jobs in the queue need to be handled
    
    jobs = queue.map do |job|
      {job_id: job.jid, job_klass: job.klass, arguments: job.args}
    end.compact;nil
    
    // if only a specific type of job needs to be triggered
    
    jobs = queue.map do |job|
          if job.klass == 'SidekiqJob'
            {job_id: job.jid, job_klass: job.klass, arguments: job.args}
          end
        end.compact;nil
    
    jobs.each do |job|
      job_found = queue.find_job(job[:job_id])
      next if job_found.nil?
      puts "JobType: #{job_found.klass}"
      puts "JobId: #{job[:job_id]}"
      puts Benchmark.realtime { job_found.klass.constantize.new.perform(*job_found.args) }
      job_found.delete
    end;nil

  
\`\`\`ruby

queue = Sidekiq::Queue.new("sidekiq_queue")

// if all jobs in the queue need to be handled

jobs = queue.map do |job|

  {job_id: job.jid, job_klass: job.klass, arguments: job.args}

end.compact;nil

// if only a specific type of job needs to be triggered

jobs = queue.map do |job|

      if job.klass == 'SidekiqJob'

        {job_id: job.jid, job_klass: job.klass, arguments: job.args}

      end

    end.compact;nil

jobs.each do |job|

  job_found = queue.find_job(job\[:job_id\])

  next if job_found.nil?

  puts "JobType: #{job_found.klass}"

  puts "JobId: #{job\[:job_id\]}"

  puts Benchmark.realtime { job_found.klass.constantize.new.perform(*job_found.args) }

  job_found.delete

end;nil  
\`\`\`